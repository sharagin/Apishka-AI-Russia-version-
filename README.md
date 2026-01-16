# Apishka.Ai API Keys

Репозиторий для работы с различными API ключами OpenRouter AI.

## Шаблон скрипта для работы с API

```python
import requests
import json
import time

# Доступные API ключи
API_KEYS = {
    "xiaomi": "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",
    "venice": "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",
    "glm_air": "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"
}

# Модели для каждого ключа
MODELS = {
    "xiaomi": "z-ai/glm-4.5-air:free",
    "venice": "z-ai/glm-4.5-air:free",
    "glm_air": "z-ai/glm-4.5-air:free"
}

def query_openrouter(key_name, prompt, max_retries=3):
    """
    Отправляет запрос к OpenRouter API
    
    Args:
        key_name: название ключа из API_KEYS
        prompt: текст запроса пользователя
        max_retries: максимальное количество попыток при ошибках
        
    Returns:
        dict: ответ от API или None при ошибке
    """
    
    if key_name not in API_KEYS:
        print(f"❌ Ключ '{key_name}' не найден. Доступные ключи: {list(API_KEYS.keys())}")
        return None
    
    api_key = API_KEYS[key_name]
    model = MODELS.get(key_name, "z-ai/glm-4.5-air:free")
    
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
        "HTTP-Referer": "https://github.com/Apishka.Ai",
        "X-Title": "Apishka.Ai API",
    }
    
    data = {
        "model": model,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ],
        "temperature": 0.7,
        "max_tokens": 1000
    }
    
    for attempt in range(max_retries):
        try:
            print(f"🔑 Используется ключ: {key_name}")
            print(f"🤖 Модель: {model}")
            print(f"📝 Запрос: {prompt[:50]}...")
            
            response = requests.post(
                url="https://openrouter.ai/api/v1/chat/completions",
                headers=headers,
                data=json.dumps(data),
                timeout=30
            )
            
            if response.status_code == 200:
                result = response.json()
                if 'choices' in result and len(result['choices']) > 0:
                    answer = result['choices'][0]['message']['content']
                    print(f"✅ Ответ получен успешно!")
                    print(f"📊 Использовано токенов: {result.get('usage', {}).get('total_tokens', 'N/A')}")
                    return {
                        "answer": answer,
                        "full_response": result,
                        "key_used": key_name,
                        "model": model
                    }
                else:
                    print(f"⚠️  Неожиданный формат ответа: {result}")
                    
            elif response.status_code == 429:
                print(f"⏳ Превышен лимит запросов для ключа '{key_name}'. Попытка {attempt + 1}/{max_retries}")
                time.sleep(2 * (attempt + 1))  # Экспоненциальная задержка
                continue
                
            else:
                print(f"❌ Ошибка API (статус {response.status_code}): {response.text}")
                if attempt < max_retries - 1:
                    time.sleep(1)
                    continue
                    
        except requests.exceptions.Timeout:
            print(f"⏱️  Таймаут запроса. Попытка {attempt + 1}/{max_retries}")
            time.sleep(1)
        except requests.exceptions.RequestException as e:
            print(f"🔌 Ошибка сети: {e}. Попытка {attempt + 1}/{max_retries}")
            time.sleep(1)
        except json.JSONDecodeError as e:
            print(f"📄 Ошибка парсинга JSON: {e}")
            break
    
    print(f"🚫 Не удалось получить ответ после {max_retries} попыток")
    return None

def try_all_keys(prompt):
    """
    Пробует все доступные ключи по очереди до получения успешного ответа
    
    Args:
        prompt: текст запроса пользователя
        
    Returns:
        dict: первый успешный ответ или None если все ключи не сработали
    """
    print(f"🔄 Пробую все доступные ключи...")
    
    for key_name in API_KEYS.keys():
        print(f"\n{'='*50}")
        print(f"🔍 Тестирую ключ: {key_name}")
        print(f"{'='*50}")
        
        result = query_openrouter(key_name, prompt, max_retries=1)
        if result:
            return result
    
    print("\n🚫 Ни один из ключей не сработал")
    return None

# Пример использования
if __name__ == "__main__":
    # Пример 1: Использование конкретного ключа
    print("Пример 1: Использование конкретного ключа")
    print("=" * 50)
    
    prompt = "What is the meaning of life?"
    result = query_openrouter("xiaomi", prompt)
    
    if result:
        print(f"\n📨 Ответ от {result['key_used']}:")
        print(f"{result['answer']}")
        print(f"\n📊 Модель: {result['model']}")
    
    # Пример 2: Перебор всех ключей
    print("\n\n" + "=" * 50)
    print("Пример 2: Перебор всех ключей до успеха")
    print("=" * 50)
    
    prompt2 = "Explain quantum computing in simple terms"
    result2 = try_all_keys(prompt2)
    
    if result2:
        print(f"\n✅ Успешный ответ получен от ключа: {result2['key_used']}")
        print(f"🤖 Модель: {result2['model']}")
        print(f"📝 Ответ: {result2['answer'][:200]}...")
