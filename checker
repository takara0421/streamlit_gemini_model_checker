import streamlit as st
import google.generativeai as genai
import time

st.title("🤖 Gemini Model Availability Checker")
st.caption("現在の環境(Streamlit Cloud等)から、実際にリクエストが通るモデルを判定します。")

api_key = st.secrets.get("GEMINI_API_KEY")
if not api_key:
    st.error("APIキーが設定されていません。secrets.tomlを確認してください。")
    st.stop()

genai.configure(api_key=api_key)

# 1. チェックしたいモデルの候補リスト
# list_models()で見えるもの + 見えないけど存在する実験的モデル(exp)を手動で追加
target_models = [
    # 安定版
    "models/gemini-1.5-flash",
    "models/gemini-1.5-pro",
    "models/gemini-1.0-pro",
    # 最新/実験的モデル (list_modelsに出てこない場合があるため明記)
    "models/gemini-2.0-flash-exp",
    "models/gemini-2.0-flash-lite", # 今回の問題児
    "models/gemini-exp-1206",
]

# 結果を保存するリスト
results = []

if st.button("全モデルの接続テストを開始"):
    progress_bar = st.progress(0)
    status_text = st.empty()
    
    for i, model_name in enumerate(target_models):
        status_text.text(f"Testing: {model_name} ...")
        
        # モデル名から "models/" 接頭辞がついている場合とついていない場合の調整
        clean_name = model_name.replace("models/", "")
        
        try:
            model = genai.GenerativeModel(clean_name)
            # 最小限の負荷でテスト
            response = model.generate_content("Hello", generation_config={"max_output_tokens": 5})
            
            # 成功
            results.append({
                "Model Name": clean_name,
                "Status": "✅ OK",
                "Note": "利用可能"
            })
            
        except Exception as e:
            error_msg = str(e)
            if "429" in error_msg and "limit: 0" in error_msg:
                note = "❌ 権限なし (Limit: 0)"
            elif "429" in error_msg:
                note = "⚠️ レート制限 (使いすぎ)"
            elif "Not Found" in error_msg or "404" in error_msg:
                note = "❓ 存在しないモデル"
            else:
                note = f"❌ エラー: {error_msg[:30]}..."

            results.append({
                "Model Name": clean_name,
                "Status": "NG",
                "Note": note
            })
        
        # プログレスバー更新
        progress_bar.progress((i + 1) / len(target_models))
        
        # 連続アクセスによるレート制限回避のため少し待機
        time.sleep(1)

    st.success("チェック完了！")
    
    # 結果を表で表示
    st.table(results)

    st.info("※ 'Limit: 0' と出るモデルは、現在の環境（無料枠×Streamlit Cloud等）ではGoogleによってブロックされています。")
