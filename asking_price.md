import requests

def get_ethusdt_ask_price(api_key):
    """
    ETH/USDT жұбы үшін ең жоғарғы сұраныс бағасын (ask) алады.
    
    Параметрлер:
        api_key (str): Ataix API кілті
    
    Қайтарады:
        float: Ask бағасы (мысалы 2850.50)
        None: Қате болған жағдайда
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

# Қолдану мысалы:
api_key = "reYvVnW1aGXS6v6ZuhPpZUDJFH4aISmGU5bHj3URhYbMMhe9hb9dxwvJ2QquEn0THDvWOedG3nIS8S7XYx2KoR"  # API кілтін ауыстырыңыз
ask_price = get_ethusdt_ask_price(api_key)

if ask_price is not None:
    print(f"ETH/USDT жұбы үшін ең жоғарғы сұраныс бағасы: {ask_price}")
else:
    print("Баға алынған жоқ")
