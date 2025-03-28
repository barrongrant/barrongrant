<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Photo Slideshow Maker</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Roboto', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f0f0;
            color: #333;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            min-height: 100vh;
            box-sizing: border-box;
        }

        header {
            background-color: #4CAF50;
            color: white;
            padding: 20px;
            text-align: center;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            width: 100%;
            box-sizing: border-box;
        }

        header h1 {
            margin: 0;
            font-size: 2.5em;
        }

        main {
            padding: 20px;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-sizing: border-box;
            width: 100%;
        }

        #imageInput {
            margin-top: 20px;
            margin-bottom: 20px;
        }

        #imageInput input[type="file"] {
            display: none;
        }

        #imageInput label {
            background-color: #007bff;
            color: white;
            padding: 10px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.3s ease;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        #imageInput label:hover {
            background-color: #0056b3;
        }

        #videoContainer {
            display: none;
            margin-top: 20px;
            border: 1px solid #ccc;
            border-radius: 5px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            overflow: hidden;
            width: 90%;
            max-width: 800px;
            aspect-ratio: 16 / 9;
        }

        #myVideo {
            width: 100%;
            height: 100%;
        }

        #generateBtn {
            margin-top: 20px;
        }

        #generateBtn button {
            background-color: #28a745;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.3s ease;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        #generateBtn button:hover {
            background-color: #218838;
        }

        #errorContainer {
            color: red;
            margin-top: 20px;
            font-size: 1em;
            display: none;
        }

        footer {
            background-color: #333;
            color: white;
            padding: 20px;
            text-align: center;
            margin-top: 40px;
            width: 100%;
            box-sizing: border-box;
            border-top: 1px solid #444;
        }

        @media (max-width: 768px) {
            header h1 {
                font-size: 2em;
            }

            main {
                padding: 10px;
            }

            #imageInput {
                margin-top: 10px;
                margin-bottom: 10px;
            }

            #generateBtn button {
                font-size: 0.9em;
                padding: 10px 18px;
            }

            footer {
                padding: 15px;
                font-size: 0.9em;
            }
        }

        @media (max-width: 480px) {
            header h1 {
                font-size: 1.75em;
            }

            nav ul li {
                margin: 0 5px;
            }

            #imageInput label {
                font-size: 0.8em;
                padding: 8px 12px;
            }

            #generateBtn button {
                font-size: 0.75em;
                padding: 8px 16px;
            }

            footer {
                font-size: 0.8em;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Photo Slideshow Maker</h1>
    </header>
    <main>
        <div id="imageInput">
            <label for="fileInput">Choose Images</label>
            <input type="file" id="fileInput" accept="image/*" multiple>
        </div>
        <div id="videoContainer">
            <video id="myVideo" controls></video>
        </div>
        <div id="generateBtn">
            <button id="createVideo">Generate Slideshow</button>
        </div>
        <div id="errorContainer">
            <p id="errorMessage"></p>
        </div>
    </main>
    <footer>
        <p>&copy; 2025 Photo Slideshow Maker. All rights reserved.</p>
    </footer>

    <script>
        const fileInput = document.getElementById('fileInput');
        const videoContainer = document.getElementById('videoContainer');
        const myVideo = document.getElementById('myVideo');
        const generateBtn = document.getElementById('createVideo');
        const errorContainer = document.getElementById('errorContainer');
        const errorMessage = document.getElementById('errorMessage');

        let imagesArray = [];
        let videoPlaying = false;

        fileInput.addEventListener('change', (event) => {
            imagesArray = Array.from(event.target.files);
            if (imagesArray.length > 0) {
                videoContainer.style.display = 'block';
                myVideo.src = URL.createObjectURL(imagesArray[0]);
                myVideo.load();
                videoPlaying = false;
            } else {
                videoContainer.style.display = 'none';
                myVideo.src = '';
            }
            errorContainer.style.display = 'none';
        });

        generateBtn.addEventListener('click', async () => { // Make the event handler async to use await
            if (imagesArray.length === 0) {
                errorMessage.textContent = "Please select images to create a slideshow.";
                errorContainer.style.display = 'block';
                return;
            }

            errorMessage.textContent = "";
            errorContainer.style.display = 'none';

            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            canvas.width = 800;
            canvas.height = 450;

            let encoder = null; // Declare encoder outside the try block
            try {
                encoder = new VideoEncoder({ // Initialize encoder
                    output: (chunk) => {
                        const blob = new Blob([chunk.data], { type: 'video/webm' });
                        const url = URL.createObjectURL(blob);

                        if (!videoPlaying) {
                            myVideo.src = url;
                            myVideo.play();
                            videoPlaying = true;
                        } else {
                            myVideo.src = url;
                            myVideo.load();
                            myVideo.play();
                        }
                    },
                    error: (err) => {
                        console.error('Encoder Error:', err);
                        errorMessage.textContent = "Error encoding video. Please try again.";
                        errorContainer.style.display = 'block';
                        videoContainer.style.display = 'none';
                    }
                });

                encoder.configure({
                    codec: 'vp8',
                    width: canvas.width,
                    height: canvas.height,
                    bitrate: 1000000,
                    framerate: 30,
                    hardwareAcceleration: 'prefer-software'
                });

                encoder.start(); // Start the encoder
            } catch (error) {
                  console.error("Failed to create encoder", error);
                  errorMessage.textContent = "Error setting up the video encoder. Your browser may not support this feature.";
                  errorContainer.style.display = 'block';
                  videoContainer.style.display = 'none';
                  return; // IMPORTANT: Exit the function if encoder setup fails
            }


            let frameCount = 0;
            const transitionDuration = 500;
            const frameRate = 30;
            let currentImageIndex = 0;
            let nextImageIndex = 1;
            let transitionStart = 0;
            let isTransitioning = false;

            async function drawFrame() { // Make drawFrame async to use await
                if (currentImageIndex >= imagesArray.length) {
                    try{
                        await encoder.flush();
                        console.log('Encoding complete!');
                     } catch(err) {
                        console.error("Error flushing encoder", err);
                        errorMessage.textContent = "Error finalizing video. Please try again.";
                        errorContainer.style.display = 'block';
                        videoContainer.style.display = 'none';
                    }
                    return;
                }

                const img = new Image();
                img.src = URL.createObjectURL(imagesArray[currentImageIndex]);
                img.onload = async () => { // Await image load
                    context.drawImage(img, 0, 0, canvas.width, canvas.height);

                    if (isTransitioning) {
                        const nextImg = new Image();
                        nextImg.src = URL.createObjectURL(imagesArray[nextImageIndex]);
                        nextImg.onload = async () => { // Await next image load
                            const elapsedTime = Date.now() - transitionStart;
                            const transitionProgress = Math.min(1, elapsedTime / transitionDuration);

                            context.globalAlpha = 1 - transitionProgress;
                            context.drawImage(img, 0, 0, canvas.width, canvas.height);

                            context.globalAlpha = transitionProgress;
                            context.drawImage(nextImg, 0, 0, canvas.width, canvas.height);
                            context.globalAlpha = 1;

                            if (elapsedTime >= transitionDuration) {
                                currentImageIndex = nextImageIndex;
                                nextImageIndex = (currentImageIndex + 1) % imagesArray.length;
                                isTransitioning = false;
                                transitionStart = 0;
                            }
                            const frame = new ImageData(new Uint8ClampedArray(context.getImageData(0, 0, canvas.width, canvas.height).data), canvas.width, canvas.height);
                            encoder.encode(frame, {
                                keyFrame: frameCount % 60 === 0
                            });
                            frameCount++;
                            requestAnimationFrame(drawFrame);
                        };
                        nextImg.onerror = (err) => {
                            console.error("Error loading next image for transition", err);
                            errorMessage.textContent = "Error loading image. Please try again.";
                            errorContainer.style.display = 'block';
                            videoContainer.style.display = 'none';
                        }
                    }
                    else {
                        const frame = new ImageData(new Uint8ClampedArray(context.getImageData(0, 0, canvas.width, canvas.height).data), canvas.width, canvas.height);
                        encoder.encode(frame, {
                            keyFrame: frameCount % 60 === 0
                        });
                        frameCount++;
                        if (frameCount % (frameRate * 2) === 0) {
                            isTransitioning = true;
                            transitionStart = Date.now();
                        }
                        requestAnimationFrame(drawFrame);
                    }
                };
                img.onerror = (err) => {
                    console.error("Error loading image", err);
                    errorMessage.textContent = "Error loading image. Please try again.";
                    errorContainer.style.display = 'block';
                    videoContainer.style.display = 'none';
                }
            }
            drawFrame();
        });
    </script>
</body>
</html>

