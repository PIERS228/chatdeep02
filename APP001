from flask import Flask, request, render_template, jsonify
import requests
import os

# 初始化 Flask 應用
app = Flask(
    __name__,
    template_folder=r"C:\Users\Piers\Desktop\PROJECT1\templates",  # 指定 templates 文件夾路徑
    static_folder=r"C:\Users\Piers\Desktop\PROJECT1\static"  # 指定 static 文件夾路徑
)

# 替換為您的 DeepSeek API 密鑰
DEEPSEEK_API_KEY = "sk-78af28495b7249e280efae4eb52a12bc"
DEEPSEEK_API_URL = "https://api.deepseek.com/v1/chat/completions"  # 確保 URL 正確

# 指定 TXT 文件路徑
TXT_FILE_PATH = r"C:\Users\Piers\Desktop\阿德勒的哲学课（共4册）.txt"

# 用於存儲分割後的文本片段
text_chunks = []

# 用於存儲用戶的對話歷史（全局變量）
conversation_history = {}


def load_and_split_text(file_path, chunk_size=10000):
    """
    加載並分割文本文件
    :param file_path: 文件路徑
    :param chunk_size: 每個片段的大小（字符數）
    :return: 分割後的文本片段列表
    """
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            text = f.read()
        # 將文本分割成較小的片段
        chunks = [text[i:i + chunk_size] for i in range(0, len(text), chunk_size)]
        return chunks
    except Exception as e:
        print(f"Error loading or splitting text: {e}")
        return []


# 在應用啟動時加載並分割文本
text_chunks = load_and_split_text(TXT_FILE_PATH)


def analyze_sentiment(text):
    """
    使用情感分析 API 分析用戶輸入的情感
    :param text: 用戶輸入的文本
    :return: 情感分析結果（如 positive, neutral, negative）
    """
    try:
        headers = {
            'Authorization': f'Bearer {DEEPSEEK_API_KEY}',
            'Content-Type': 'application/json'
        }
        data = {
            "model": "sentiment-analysis-model",  # 替換為實際的情感分析模型
            "inputs": text
        }
        response = requests.post("https://api.deepseek.com/v1/sentiment", headers=headers, json=data)
        response.raise_for_status()  # 檢查 HTTP 錯誤
        return response.json().get('sentiment', 'neutral')
    except requests.exceptions.RequestException as e:
        print(f"Sentiment analysis failed: {e}")
        return 'neutral'


def get_psychological_exercise(sentiment):
    """
    根據用戶情感返回合適的心理練習
    :param sentiment: 用戶情感（如 positive, neutral, negative）
    :return: 心理練習內容
    """
    exercises = {
        'negative': "我們可以試著做一個小練習：寫下你最近感到困擾的三件事，然後思考它們背後是否有共同的原因。",
        'neutral': "你可以試著每天記錄一件讓你感到感激的小事，這有助於提升你的幸福感。",
        'positive': "你現在的情緒似乎不錯，可以試著回顧一下最近的成功經驗，並思考是什麼讓你感到滿足。"
    }
    return exercises.get(sentiment, "讓我們一起探索你的內心世界。")


@app.route('/')
def index():
    # 渲染 index.html 模板
    return render_template('index.html')


@app.route('/chat', methods=['POST'])
def chat():
    # 獲取用戶輸入的消息
    user_message = request.json.get('message')

    # 分析用戶情感
    sentiment = analyze_sentiment(user_message)

    # 獲取心理練習
    exercise = get_psychological_exercise(sentiment)

    # 獲取用戶的唯一標識（例如 IP 地址或用戶名）
    user_id = request.remote_addr  # 使用 IP 地址作為用戶標識

    # 初始化用戶的對話歷史
    if user_id not in conversation_history:
        conversation_history[user_id] = []

    # 將用戶的輸入添加到對話歷史
    conversation_history[user_id].append({"role": "user", "content": user_message})

    # 構建系統提示
    system_message = "你是阿德勒，一位心理醫生。你語氣溫和、耐心，善於傾聽並提供專業的心理建議。\n"
    system_message += f"{exercise}\n"  # 加入心理練習
    system_message += "以下是文本的上下文：\n" + "\n".join(text_chunks[:5])

    # 將對話歷史作為上下文傳遞給 AI
    messages = [{"role": "system", "content": system_message}] + conversation_history[user_id]

    data = {
        "model": "deepseek-chat",
        "messages": messages,
        "max_tokens": 1500
    }

    try:
        # 發送請求到 DeepSeek API
        headers = {
            'Authorization': f'Bearer {DEEPSEEK_API_KEY}',
            'Content-Type': 'application/json'
        }
        response = requests.post(DEEPSEEK_API_URL, headers=headers, json=data)
        response.raise_for_status()  # 檢查 HTTP 錯誤

        # 假設 API 返回的 JSON 結構中有 'choices' 字段
        ai_response = response.json().get('choices')[0].get('message').get('content').strip()

        # 將 AI 的回應添加到對話歷史
        conversation_history[user_id].append({"role": "assistant", "content": ai_response})

        return jsonify({'message': ai_response})
    except requests.exceptions.RequestException as e:
        # 捕獲並打印異常信息
        print(f"API request failed: {e}")
        return jsonify({'message': '抱歉，我暫時無法處理你的請求。請稍後再試。'}), 500


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
