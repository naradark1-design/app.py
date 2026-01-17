# app.py

youtube-ai-streamlit/
├─ app.py                ✅ Streamlit Cloud용
├─ requirements.txt
└─ .streamlit/
   └─ secrets.toml


import streamlit as st
import openai, re
from youtube_transcript_api import YouTubeTranscriptApi

# ===============================
# 기본 설정
# ===============================
st.set_page_config(
    page_title="Personal YouTube AI",
    page_icon="🎥",
    layout="wide"
)

openai.api_key = st.secrets["OPENAI_API_KEY"]

# ===============================
# CSS
# ===============================
st.markdown("""
<style>
body { background-color: #0e1117; color: #fafafa; }
.card {
    background-color: #161b22;
    padding: 25px;
    border-radius: 16px;
    margin-bottom: 20px;
}
.hero {
    padding: 30px;
    border-radius: 20px;
    background: linear-gradient(135deg,#1f2933,#111827);
    margin-bottom: 30px;
}
</style>
""", unsafe_allow_html=True)

# ===============================
# 유틸
# ===============================
def extract_video_id(url):
    m = re.search(r"(?:v=|\/)([0-9A-Za-z_-]{11})", url)
    return m.group(1) if m else None

def get_transcript(video_id):
    t = YouTubeTranscriptApi.get_transcript(video_id, ["ko","en"])
    return " ".join(x["text"] for x in t)

def chat(prompt, tokens=500):
    r = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[{"role":"user","content":prompt}],
        temperature=0.3,
        max_tokens=tokens
    )
    return r.choices[0].message["content"]

# ===============================
# Sidebar
# ===============================
menu = st.sidebar.radio("MENU", ["🏠 Home", "ℹ️ About"])

# ===============================
# HOME
# ===============================
if menu == "🏠 Home":

    st.markdown("""
    <div class="hero">
        <h1>🎥 Personal YouTube AI</h1>
        <p>유튜브 영상 하나로 요약 · 질문 · 콘텐츠 생성을 합니다</p>
    </div>
    """, unsafe_allow_html=True)

    col1, col2 = st.columns([4,1])
    with col1:
        url = st.text_input("YouTube URL")
    with col2:
        run = st.button("🚀 분석", use_container_width=True)

    question = st.text_input("질문 (선택)")

    if run and url:
        vid = extract_video_id(url)
        if not vid:
            st.error("유효한 YouTube URL이 아닙니다.")
            st.stop()

        with st.spinner("자막 분석 중..."):
            transcript = get_transcript(vid)

        summary = chat(f"다음 자막을 요약해줘:\n{transcript}")
        keywords = chat(f"핵심 키워드 10개:\n{transcript}", 150)
        blog = chat(f"블로그 글 작성:\n{transcript}")
        shorts = chat(f"쇼츠용 30초 스크립트:\n{transcript}", 200)

        st.markdown('<div class="card"><h2>📌 요약</h2>', unsafe_allow_html=True)
        st.write(summary)
        st.markdown('</div>', unsafe_allow_html=True)

        t1, t2, t3 = st.tabs(["🏷 키워드", "🎬 쇼츠", "✍️ 블로그"])
        with t1: st.info(keywords)
        with t2: st.write(shorts)
        with t3: st.write(blog)

        if question:
            answer = chat(f"자막 기반 질문 답변:\n질문:{question}\n자막:{transcript}")
            st.markdown('<div class="card"><h2>💬 답변</h2>', unsafe_allow_html=True)
            st.write(answer)
            st.markdown('</div>', unsafe_allow_html=True)

# ===============================
# ABOUT
# ===============================
else:
    st.markdown("""
    ## About
    개인 전용 YouTube 분석 AI입니다.
    """)


streamlit
openai
youtube-transcript-api
