<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="theme-color" content="#0b0b0b">
  <title>PALM GOLD</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 20px;
      background: #0b0b0b;
      color: #f5e8c8;
      font-family: Arial, sans-serif;
      text-align: center;
    }

    .container {
      max-width: 600px;
      margin: 0 auto;
    }

    .logo {
      color: #d8b56a;
      letter-spacing: 5px;
      font-size: 18px;
      font-weight: bold;
      margin-top: 25px;
    }

    h1 {
      margin-top: 55px;
      font-family: Georgia, serif;
      font-size: clamp(35px, 8vw, 52px);
      color: #e2c27b;
    }

    button {
      width: 100%;
      padding: 16px;
      margin-top: 15px;
      border: 1px solid #d8b56a;
      border-radius: 10px;
      background: linear-gradient(135deg, #f0d99a, #bd9143);
      color: #111;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }

    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    .card {
      margin-top: 35px;
      padding: 22px;
      border: 1px solid #594725;
      border-radius: 18px;
      background: #151515;
    }

    video,
    #preview {
      display: none;
      width: 100%;
      margin-top: 20px;
      border-radius: 12px;
      background: #000;
    }

    input,
    textarea {
      display: block;
      width: 100%;
      margin-top: 12px;
      padding: 13px;
      background: #090909;
      border: 1px solid #594725;
      border-radius: 8px;
      color: white;
      font-size: 15px;
    }

    label {
      display: block;
      text-align: left;
      margin-top: 20px;
      color: #d8b56a;
      font-size: 13px;
    }

    #status {
      margin-top: 15px;
      color: #d8b56a;
      font-size: 13px;
      line-height: 1.7;
      white-space: pre-wrap;
    }

    footer {
      margin-top: 35px;
      color: #756b58;
      font-size: 11px;
    }
  </style>
</head>

<body>
  <main class="container">
    <div class="logo">PALM GOLD</div>

    <h1>Your Hands.<br>Your Story.</h1>

    <section class="card">
      <h2>手相占いする</h2>

      <button id="start" type="button">
        📷 手相を占う
      </button>

      <video id="video" autoplay playsinline></video>
      <canvas id="canvas" hidden></canvas>
      <img id="preview" alt="撮影した手の写真">

      <button id="take" type="button" hidden>
        写真を撮る
      </button>

      <button id="retake" type="button" hidden>
        撮り直す
      </button>

      <label for="nickname">ニックネーム（10字以内）</label>
      <input
        id="nickname"
        type="text"
        maxlength="10"
        placeholder="ニックネームを入力"
      >

      <label for="question">相談内容（100字以内）</label>
      <textarea
        id="question"
        rows="3"
        maxlength="100"
        placeholder="相談内容を入力"
      ></textarea>

      <button id="send" type="button" disabled>
        Telegramに送信する →
      </button>

      <div id="status" role="status" aria-live="polite"></div>
    </section>

    <footer>PALM GOLD · YOUR STORY</footer>
  </main>

  <script>
    const video = document.getElementById("video");
    const canvas = document.getElementById("canvas");
    const preview = document.getElementById("preview");
    const start = document.getElementById("start");
    const take = document.getElementById("take");
    const retake = document.getElementById("retake");
    const send = document.getElementById("send");
    const status = document.getElementById("status");
    const nickname = document.getElementById("nickname");
    const question = document.getElementById("question");

    let stream = null;
    let photo = null;
    let previewUrl = null;

    function stopCamera() {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
        stream = null;
      }

      video.srcObject = null;
    }

    function clearPreview() {
      if (previewUrl) {
        URL.revokeObjectURL(previewUrl);
        previewUrl = null;
      }

      preview.removeAttribute("src");
      preview.style.display = "none";
    }

    start.addEventListener("click", async () => {
      try {
        status.textContent = "カメラを起動しています…";

        if (!navigator.mediaDevices?.getUserMedia) {
          throw new Error(
            "カメラを使うにはHTTPS対応のサイトで開いてください。"
          );
        }

        stopCamera();
        clearPreview();

        photo = null;
        send.disabled = true;
        retake.hidden = true;

        stream = await navigator.mediaDevices.getUserMedia({
          video: {
            facingMode: { ideal: "environment" }
          },
          audio: false
        });

        video.srcObject = stream;
        video.style.display = "block";
        take.hidden = false;

        status.textContent =
          "手のひら全体が映るようにカメラを向けてください。";

      } catch (error) {
        stopCamera();
        status.textContent =
          "カメラを起動できませんでした。\n" + error.message;
      }
    });

    take.addEventListener("click", () => {
      if (!video.videoWidth) {
        status.textContent = "カメラの準備ができていません。";
        return;
      }

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      canvas.getContext("2d").drawImage(
        video, 0, 0, canvas.width, canvas.height
      );

      canvas.toBlob(blob => {
        if (!blob) {
          status.textContent = "写真を撮影できませんでした。";
          return;
        }

        photo = blob;
        clearPreview();

        previewUrl = URL.createObjectURL(blob);
        preview.src = previewUrl;
        preview.style.display = "block";

        video.style.display = "none";
        take.hidden = true;
        retake.hidden = false;
        send.disabled = false;

        stopCamera();

        status.textContent =
          "写真を確認してから、送信ボタンを押してください。";
      }, "image/jpeg", 0.9);
    });

    retake.addEventListener("click", () => {
      photo = null;
      send.disabled = true;
      retake.hidden = true;
      clearPreview();
      start.click();
    });

    send.addEventListener("click", async () => {
      if (!photo) {
        status.textContent = "先に手の写真を撮影してください。";
        return;
      }

      if (nickname.value.length > 10) {
        status.textContent = "ニックネームは10字以内にしてください。";
        return;
      }

      if (question.value.length > 100) {
        status.textContent = "相談内容は100字以内にしてください。";
        return;
      }

      send.disabled = true;
      status.textContent = "送信しています…";

      const form = new FormData();
      form.append("photo", photo, "palm.jpg");
      form.append("nickname", nickname.value);
      form.append("question", question.value);

      try {
        const response = await fetch("/api/send", {
          method: "POST",
          body: form
        });

        const result = await response.json();

        if (!response.ok) {
          throw new Error(
            result.error || "送信に失敗しました。"
          );
        }

        status.textContent =
          "送信完了！鑑定をお待ちください。";

      } catch (error) {
        status.textContent =
          "送信できませんでした。\n" +
          "Telegram送信用のサーバー設定が必要です。";

        send.disabled = false;
      }
    });

    window.addEventListener("pagehide", stopCamera);
  </script>
</body>
</html>
