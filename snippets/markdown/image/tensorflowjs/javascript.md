Learn more about how to use the code snippet on [github](https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image).

```html
<div>Teachable Machine Image Model</div>
<button type="button" onclick="init()">Start</button>
<div id="webcam-container"></div>
<div id="label-container"></div>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
<script type="text/javascript">
    // More API functions here:
    // https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image

    // the link to your model provided by Teachable Machine export panel
    const URL = "{{URL}}";

    let model, webcam, labelContainer, maxPredictions;

    // Load the image model and setup the webcam
    async function init() {
        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";

        // load the model and metadata
        // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
        // or files from your local hard drive
        // Note: the pose library adds "tmImage" object to your window (window.tmImage)
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();

        // Convenience function to setup a webcam
        const flip = true; // whether to flip the webcam
        webcam = new tmImage.Webcam(200, 200, flip); // width, height, flip
        await webcam.setup(); // request access to the webcam
        await webcam.play();
        window.requestAnimationFrame(loop);

        // append elements to the DOM
        document.getElementById("webcam-container").appendChild(webcam.canvas);
        labelContainer = document.getElementById("label-container");
        for (let i = 0; i < maxPredictions; i++) { // and class labels
            labelContainer.appendChild(document.createElement("div"));
        }
    }

    async function loop() {
        webcam.update(); // update the webcam frame
        await predict();
        window.requestAnimationFrame(loop);
    }

    // run the webcam image through the image model
   const predictions = await model.predict(webcam.canvas);
  predictions.sort((a, b) => b.probability - a.probability);

  const best = predictions[0];
  const name = best.className.trim();
  const confidence = best.probability;

  const result = document.getElementById("result");
  const guide = document.getElementById("guide");

  if (confidence < 0.65) {
    result.textContent = "종류를 구별하기 어려워요.";
    guide.textContent =
      "약을 카메라 중앙에 놓고 다시 비춰 주세요.";
    return;
  }

  if (name.includes("빈 화면")) {
    result.textContent = "폐의약품을 비춰 주세요.";
    guide.textContent = "카메라 앞에 약을 놓아 주세요.";

  } else if (name.includes("알약")) {
    result.textContent = "💊 알약이 감지되었습니다!";
    guide.textContent =
      "첫 번째 수거함에 넣어 주세요!";

  } else if (name.includes("물약")) {
    result.textContent = "🧴 물약이 감지되었습니다!";
    guide.textContent =
      "두 번째 수거함에 넣어 주세요!";

  } else if (name.includes("가루약")) {
    result.textContent = "📦 가루약이 감지되었습니다!";
    guide.textContent =
      "세 번째 수거함에 넣어 주세요!";

  } else {
    result.textContent = "종류를 확인해 주세요.";
    guide.textContent =
      "약국이나 담당자에게 배출 방법을 문의해 주세요.";
  }

  result.textContent +=
    " (AI 예측 신뢰도: " +
    (confidence * 100).toFixed(1) + "%)";
}
```
