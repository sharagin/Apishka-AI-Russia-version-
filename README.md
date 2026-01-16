# Apishka.Ai API Keys

## Список ключей и соответствующих моделей

| Название ключа | API Key | Модель (model) |
|----------------|---------|----------------|
| **xiaomi** | `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` | `xiaomi/mimo-v2-flash:free` |
| **venice** | `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` | `cognitivecomputations/dolphin-mistral-24b-venice-edition:free` |
| **glm** | `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` | `z-ai/glm-4.5-air:free` |

## Шаблон скрипта для работы с API (Python)

```python
import requests
import json

# ВЫБЕРИТЕ ОДИН КЛЮЧ И МОДЕЛЬ ИЗ ТАБЛИЦЫ ВЫШЕ:

# Для Xiaomi Mimo V2 Flash:
API_KEY = "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"
MODEL = "xiaomi/mimo-v2-flash:free"

# Для Venice Uncersoored:
# API_KEY = "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"
# MODEL = "cognitivecomputations/dolphin-mistral-24b-venice-edition:free"

# Для GLM 4.5 Air:
# API_KEY = "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"
# MODEL = "z-ai/glm-4.5-air:free"

# Настройки сайта (опционально)
YOUR_SITE_URL = "https://your-site.com"
YOUR_SITE_NAME = "Your Site"

def query_ai(prompt):
    """
    Отправляет запрос к OpenRouter AI
    
    Args:
        prompt (str): Текст запроса пользователя
        
    Returns:
        str: Ответ от AI или сообщение об ошибке
    """
    
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json",
        "HTTP-Referer": YOUR_SITE_URL,  # Опционально
        "X-Title": YOUR_SITE_NAME,      # Опционально
    }
    
    data = {
        "model": MODEL,  # Модель зависит от выбранного ключа
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ]
    }
    
    try:
        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers=headers,
            data=json.dumps(data),
            timeout=30
        )
        
        if response.status_code == 200:
            result = response.json()
            return result['choices'][0]['message']['content']
        else:
            return f"Ошибка API: {response.status_code} - {response.text}"
            
    except Exception as e:
        return f"Ошибка соединения: {str(e)}"

# Пример использования
if __name__ == "__main__":
    # Ваш запрос
    user_prompt = "What is the meaning of life?"
    
    # Получение ответа
    answer = query_ai(user_prompt)
    
    # Вывод результата
    print("📝 Запрос:", user_prompt)
    print("🤖 Ответ:", answer)
    print("🔑 Использован ключ:", MODEL.split('/')[0])  # Показывает название модели
    print("🚀 Модель:", MODEL)
