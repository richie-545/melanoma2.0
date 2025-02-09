const modelPath = 'skin_lesion_web_model/model.json';
let model;

async function loadModel() {
    model = await tf.loadLayersModel(modelPath);
    console.log("Model Loaded!");
}

document.getElementById('file-input').addEventListener('change', async function (event) {
    const file = event.target.files[0];
    if (!file) return;

    const img = document.createElement('img');
    img.src = URL.createObjectURL(file);
    img.onload = async function () {
        const tensor = preprocessImage(img);
        const prediction = await model.predict(tensor).data();
        displayResult(prediction);
    };
});

function preprocessImage(image) {
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    canvas.width = 100;
    canvas.height = 100;
    ctx.drawImage(image, 0, 0, 100, 100);

    let imgData = tf.browser.fromPixels(canvas).toFloat();
    imgData = imgData.div(255.0).expandDims();
    return imgData;
}

function displayResult(prediction) {
    const resultText = prediction[0] > prediction[1] ? "Malignant" : "Benign";
    document.getElementById('result').innerText = `Prediction: ${resultText}`;
}

// Load model when page starts
loadModel();
