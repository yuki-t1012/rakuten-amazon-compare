# rakuten-amazon-compare
import streamlit as st
import requests
from bs4 import BeautifulSoup
import csv
import time

# 楽天APIで商品を探す
def fetch_rakuten_items(keyword, api_key, hits=100, genre_id=None):
    url = "https://app.rakuten.co.jp/services/api/IchibaItem/Search/20170706"
    params = {
        "applicationId": api_key,
        "keyword": keyword,
        "hits": min(hits, 100),
        "format": "json"
    }
    if genre_id:
        params["genreId"] = genre_id

    response = requests.get(url, params=params)
    items = response.json().get("Items", [])
    return [{
        "name": item["Item"]["itemName"],
        "price": item["Item"]["itemPrice"],
        "url": item["Item"]["itemUrl"]
    } for item in items]

# Amazonで商品価格を調べる（ざっくり）
def fetch_amazon_price(keyword):
    headers = {
        "User-Agent": "Mozilla/5.0"
    }
    url = f"https://www.amazon.co.jp/s?k={keyword}"
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.text, "html.parser")
    price_span = soup.find("span", {"class": "a-offscreen"})
    if price_span:
        price_text = price_span.text.replace("￥", "").replace(",", "")
        try:
            return int(float(price_text))
        except:
            return None
    return None

# 結果をCSVで保存
def export_to_csv(items, filename="output.csv"):
    with open(filename, mode="w", newline="", encoding="utf-8") as file:
        writer = csv.writer(file)
        writer.writerow(["商品名", "楽天価格", "Amazon価格", "商品URL"])
        for item in items:
            writer.writerow([item["name"], item["rakuten_price"], item["amazon_price"], item["url"]])
    return filename

# Streamlitで表示
def main():
    st.title("楽天 vs Amazon 価格比較ツール")
    st.markdown("キーワードを入力すると楽天の商品を検索して、Amazonと価格を比較します。")

    api_key = st.text_input("楽天APIキー", type="password")
    keyword = st.text_input("検索ワード", value="ワイヤレスイヤホン")
    genre_id = st.text_input("楽天ジャンルID（任意）", value="")
