# Seal-seed-5
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>戦力集計ツール</title>
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4"></script>
  <style>
    body {
      background-color: black;
      color: white;
      font-family: sans-serif;
      max-width: 600px;
      margin: 40px auto;
      text-align: center;
    }
    #log {
      white-space: pre-wrap;
      text-align: left;
      margin-top: 20px;
      font-size: 14px;
      color: #ccc;
    }
    #result {
      margin-top: 20px;
      font-size: 20px;
      font-weight: bold;
    }
    input[type="file"] {
      margin-top: 20px;
      color: #fff;
    }
    h1, p {
      margin: 0;
      padding: 0;
    }
    p {
      margin-top: 10px;
      font-size: 16px;
    }
  </style>
</head>
<body>

  <h1>スクショでラクラク！戦力統計sealseed</h1>
  <p>画像を複数選択して戦力の合計を出せるよ！</p>
  <input type="file" id="imageInput" accept="image/*" multiple>
  
  <div id="log">ログ：待機中…</div>
  <div id="result">結果：--</div>

  <script>
    window.addEventListener('DOMContentLoaded', () => {
      const input = document.getElementById('imageInput');
      const log   = document.getElementById('log');
      const result= document.getElementById('result');

      input.addEventListener('change', async () => {
        const files = Array.from(input.files);
        if (files.length === 0) {
          result.textContent = '結果：画像を選択してください';
          return;
        }

        log.textContent = `ログ：処理開始 (${files.length} 枚)\n`;
        let total = 0;
        let allValues = [];

        for (const file of files) {
          log.textContent += `OCR 中 → ${file.name}\n`;
          try {
            const { data: { text } } = await Tesseract.recognize(file, 'jpn+eng');
            console.log(`OCR raw (${file.name}):\n`, text);

            // ゆるめの抽出条件＋範囲指定で誤検出対策
            const values = (text.match(/\d[\d,]{5,}/g) || [])  // カンマ付きの6桁以上
              .map(v => parseInt(v.replace(/,/g, ''), 10))
              .filter(v => v >= 1000000 && v < 999999999); // 常識的な範囲に限定

            log.textContent += ` → 抽出値 [${values.join(', ')}]\n`;
            allValues.push(...values);
            total += values.reduce((sum, v) => sum + v, 0);

          } catch (e) {
            console.error(e);
            log.textContent += ` → エラー発生\n`;
          }
        }

        result.textContent = 
          `結果：抽出値 → ${allValues.join(', ')}\n合計 → ${total.toLocaleString()}`;
        log.textContent += '\nログ：処理完了';
      });
    });
  </script>
</body>
</html>
