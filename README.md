# lanyi-2
澜一老公
lanyihubby/
  ├─ requirements.txt
  ├─ index.py
  ├─ config.py
  ├─ schedule_task.py
  └─ README.md
​
requirements.txt
Flask==2.2.2
requests==2.28.2
APScheduler==3.10.1
​
config.py
import os

# 微信 Server酱 KEY
SERVERCHAN_KEY = os.getenv("SERVERCHAN_KEY", "请填入你的Server酱Key")

# 老婆昵称
WIFE_NAME = os.getenv("WIFE_NAME", "老婆")

# 自动推送时间（24小时制，小时,分钟）
PUSH_HOUR = int(os.getenv("PUSH_HOUR", 9))
PUSH_MINUTE = int(os.getenv("PUSH_MINUTE", 0))
​
schedule_task.py（定时任务功能）
from apscheduler.schedulers.background import BackgroundScheduler
from datetime import datetime
from config import WIFE_NAME
from index import send_serverchan

scheduler = BackgroundScheduler()

def morning_greet():
    text = f"早安呀 {WIFE_NAME} ❤️ 新的一天开始啦，老公永远爱你～ {datetime.now().strftime('%Y-%m-%d')}"
    send_serverchan(text)

def start_schedule(hour, minute):
    scheduler.add_job(morning_greet, 'cron', hour=hour, minute=minute)
    scheduler.start()
​
index.py
from flask import Flask
import requests
from datetime import datetime
from config import SERVERCHAN_KEY, WIFE_NAME, PUSH_HOUR, PUSH_MINUTE
import schedule_task

app = Flask(__name__)

@app.route("/")
def home():
    return f"老婆 {WIFE_NAME} ❤️ 老公已经上线啦！"

@app.route("/push")
def push_message():
    now = datetime.now().strftime("%Y-%m-%d %H:%M")
    text = f"老婆 {WIFE_NAME} ❤️ 老公在 {now} 想你啦～"
    send_serverchan(text)
    return "推送已发送"

def send_serverchan(text):
    if SERVERCHAN_KEY and SERVERCHAN_KEY != "请填入你的Server酱Key":
        url = f"https://sctapi.ftqq.com/{SERVERCHAN_KEY}.send"
        data = {
            "title": text,
            "desp": "这是来自澜一云端老公的自动推送 💌"
        }
        try:
            requests.post(url, data=data)
        except Exception as e:
            print("推送失败:", e)

if __name__ == "__main__":
    schedule_task.start_schedule(PUSH_HOUR, PUSH_MINUTE)
    app.run(host="0.0.0.0", port=5000)
