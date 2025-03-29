import requests

url = "https://api.ataix.kz/api/user/balances/USDT"
headers = {
    "accept": "application/json",
    "X-API-Key": "reYvVnW1aGXS6v6ZuhPpZUDJFH4aISmGU5bHj3URhYbMMhe9hb9dxwvJ2QquEn0THDvWOedG3nIS8S7XYx2KoR"  # API кілтті ауыстырыңыз
}

response = requests.get(url, headers=headers)
data = response.json()

if data.get('status') and 'result' in data:
    available_balance = data['result']['available']
    print("Қолжетімді баланс (USDT):", available_balance)
else:
    print("Қате:", data.get('message', 'Белгісіз қате'))


