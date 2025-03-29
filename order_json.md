import requests
import json
from datetime import datetime

def get_ethusdt_ask_price(api_key):
    """
    ETH/USDT жұбы үшін ең жоғарғы сұраныс бағасын (ask) алады.
    """
    url = "https://api.ataix.kz/api/symbols"
    headers = {
        "accept": "application/json",
        "X-API-Key": api_key
    }

    try:
        response = requests.get(url, headers=headers)
        data = response.json()
        
        if data.get("status") and isinstance(data.get("result"), list):
            for symbol in data["result"]:
                if symbol.get("symbol") == "ETH/USDT":
                    return float(symbol["ask"]) if symbol.get("ask") else None
            print("ETH/USDT жұбы табылмады")
            return None
        else:
            print("Қате:", data.get("message", "API жауабы қате"))
            return None
    except Exception as e:
        print("API-запрос қатесі:", e)
        return None

def place_order(api_key, symbol, side, price, quantity):
    """
    Ордер жасау және файлға сақтау
    """
    url = "https://api.ataix.kz/api/orders"
    headers = {
        "accept": "application/json",
        "X-API-Key": api_key,
        "Content-Type": "application/json"
    }
    data = {
        "symbol": symbol,
        "side": side,
        "type": "limit",
        "quantity": quantity,
        "price": price,
        "subType": "gtc"
    }

    try:
        response = requests.post(url, headers=headers, json=data)
        result = response.json()
        
        if result.get("status"):
            # Ордерді файлға сақтау
            save_order_to_file({
                "id": result.get("result", {}).get("id"),
                "symbol": symbol,
                "side": side,
                "price": price,
                "quantity": quantity,
                "status": "NEW",
                "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            })
        
        return result
    except Exception as e:
        print("Ордер қатесі:", e)
        return None

def save_order_to_file(order_data):
    """Ордерді JSON файлына сақтау"""
    try:
        # Файлды оқу (егер бар болса)
        try:
            with open("orders.json", "r") as f:
                orders = json.load(f)
        except (FileNotFoundError, json.JSONDecodeError):
            orders = []
        
        # Жаңа ордерді қосу
        orders.append(order_data)
        
        # Файлға жазу
        with open("orders.json", "w") as f:
            json.dump(orders, f, indent=2)
            
        print(f"Ордер {order_data['id']} файлға сақталды")
    except Exception as e:
        print("Файлға сақтау қатесі:", e)

# Қолдану мысалы:
api_key = "reYvVnW1aGXS6v6ZuhPpZUDJFH4aISmGU5bHj3URhYbMMhe9hb9dxwvJ2QquEn0THDvWOedG3nIS8S7XYx2KoR"

# 1. Ағымдағы бағаны алу
ask_price = get_ethusdt_ask_price(api_key)

if ask_price is not None:
    print(f"ETH/USDT жұбы үшін ең жоғарғы сұраныс бағасы: {ask_price}")
    
    # 2. Ордер параметрлерін есептеу
    order_price = round(ask_price * 0.98, 2)  # 2% төмен
    quantity = 0.0001  # Ордер көлемі
    
    # 3. Ордер жасау
    order_result = place_order(
        api_key=api_key,
        symbol="ETH/USDT",
        side="buy",
        price=order_price,
        quantity=quantity
    )
    
    if order_result and order_result.get("status"):
        print(f"Ордер сәтті жасалды! ID: {order_result.get('result', {}).get('id')}")
    else:
        print("Ордер жасау сәтсіз аяқталды:", order_result.get("message", "Белгісіз қате"))
else:
    print("Баға алынған жоқ, ордер жасалмады")
