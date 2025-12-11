import gradio as gr
import speech_recognition as sr
import time

# -------------------------------
# API محاكي داخلي (محاكاة تنفيذ الطلب)
# -------------------------------
def fake_api(service_name, user_message):
    time.sleep(0.5)  # محاكاة معالجة الطلب
    return f"🔧 API (محاكاة):\nتم تنفيذ خدمة {service_name} بنجاح.\nالطلب: {user_message}\nالحالة: مكتمل ✅"

# -------------------------------
# تحديد الخدمة بناءً على النص
# -------------------------------
def detect_service(msg):
    msg = msg.lower()
    if "جواز" in msg:
        return "تجديد الجواز"
    if "هوية" in msg:
        return "تجديد الهوية"
    if "مخالفة" in msg:
        return "الاستعلام عن المخالفات"
    if "تأشيرة" in msg:
        return "إصدار تأشيرة"
    if "تابع" in msg:
        return "إضافة تابع"
    return "خدمة غير معروفة"

# -------------------------------
# تحويل الصوت إلى نص
# -------------------------------
def speech_to_text(audio_file):
    if audio_file is None:
        return ""
    recognizer = sr.Recognizer()
    with sr.AudioFile(audio_file) as source:
        audio_data = recognizer.record(source)
    try:
        return recognizer.recognize_google(audio_data, language="ar-SA")
    except:
        return "لم أستطع التعرف على الصوت"

# -------------------------------
# التشات بوت
# -------------------------------
def chatbot(message, history):
    service = detect_service(message)
    api_response = fake_api(service, message)
    bot_reply = f"📩 تم إرسال طلبك.\n{api_response}"
    history.append((f"أنت: {message}", f"أبشر الذكي: {bot_reply}"))
    return history, ""

# -------------------------------
# واجهة أبشر المحاكية
# -------------------------------
css_style = """
#header { background:#0b8a3e; color:white; font-size:28px; font-weight:bold;
          padding:20px; text-align:center; border-radius:10px 10px 0 0;}
#services { background:#e7f5ec; padding:15px; border-radius:10px;
            border:1px solid #0b8a3e; }
#chatbox { border:2px solid #0b8a3e; border-radius:10px; padding:10px;
           background:white; height:350px; overflow:auto; }
.service-btn { width:100%; margin-bottom:5px; text-align:right; }
"""

with gr.Blocks(css=css_style) as demo:
    # الهيدر
    gr.HTML("<div id='header'>منصة أبشر – النموذج الذكي (محاكاة)</div>")

    with gr.Row():
        # القائمة الجانبية
        with gr.Column(scale=1):
            gr.HTML("""
                <div id='services'>
                <h4>الخدمات</h4>
                </div>
            """)
            btn1 = gr.Button("🟢 تجديد الجواز", elem_classes="service-btn")
            btn2 = gr.Button("🟢 تجديد الهوية", elem_classes="service-btn")
            btn3 = gr.Button("🟢 الاستعلام عن المخالفات", elem_classes="service-btn")
            btn4 = gr.Button("🟢 إصدار التأشيرة", elem_classes="service-btn")
            btn5 = gr.Button("🟢 إضافة تابع", elem_classes="service-btn")

        # صندوق التشات
        with gr.Column(scale=3):
            chatbot_box = gr.Chatbot(label="أبشر الذكي", elem_id="chatbox")
            user_input = gr.Textbox(label="اكتب طلبك هنا")
            mic = gr.Audio(type="filepath", label="🎤 تسجيل صوتي")
            btn_text = gr.Button("إرسال")
            btn_voice = gr.Button("تحويل الصوت إلى نص")

            # إرسال النص
            btn_text.click(fn=chatbot, inputs=[user_input, chatbot_box], outputs=[chatbot_box, user_input])
            # تحويل الصوت
            btn_voice.click(fn=speech_to_text, inputs=[mic], outputs=[user_input])

            # ربط الأزرار الجانبية لإرسال نص تلقائي
            btn1.click(fn=lambda: chatbot("أبغى أجدد الجواز", chatbot_box.value),
                       inputs=[], outputs=[chatbot_box, user_input])
            btn2.click(fn=lambda: chatbot("أبغى أجدد الهوية", chatbot_box.value),
                       inputs=[], outputs=[chatbot_box, user_input])
            btn3.click(fn=lambda: chatbot("أبغى استعلم عن المخالفات", chatbot_box.value),
                       inputs=[], outputs=[chatbot_box, user_input])
            btn4.click(fn=lambda: chatbot("أبغى أصدر تأشيرة", chatbot_box.value),
                       inputs=[], outputs=[chatbot_box, user_input])
            btn5.click(fn=lambda: chatbot("أبغى أضيف تابع", chatbot_box.value),
                       inputs=[], outputs=[chatbot_box, user_input])

demo.launch()
