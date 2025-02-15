<script>
    import { onMount } from 'svelte';

    let imageSrc = '';
    let canvas, imageSelect, shareBtn, selfieBooth;
  
    // Function to handle image selection
    function handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          imageSrc = reader.result;
          drawPoster();
        };
        reader.readAsDataURL(file);
      }
    }
  
    // Draw the poster with the uploaded image
    function drawPoster() {
        const ctx = canvas.getContext('2d');

        // Clear canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Draw background template
        const template = new Image();
        template.src = 'img/template.png'; // Replace with actual template path
        template.onload = () => {
            ctx.drawImage(template, 0, 0, canvas.width, canvas.height);

            // Draw user image as rounded
            if (imageSrc) {
                const userImage = new Image();
                userImage.src = imageSrc;
                userImage.onload = () => {
                    
                    let imageSize = resizeToMax200(userImage.width, userImage.height);

                    const imgWidth = imageSize.width; // Example size
                    const imgHeight = imageSize.height;
                    const centerX = canvas.width / 2;
                    const centerY = canvas.height / 2;

                    // Create a circular clipping path
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(centerX, centerY, imgWidth / 2, 0, Math.PI * 2);
                    ctx.closePath();
                    ctx.clip();

                    // Draw the image inside the circular region
                    ctx.drawImage(
                        userImage,
                        centerX - imgWidth / 2,
                        centerY - imgHeight / 2,
                        imgWidth,
                        imgHeight
                    );

                    ctx.restore(); // Restore clipping path

                    canvas.classList.remove('fixed');
                    imageSelect.classList.add('hidden');

                    shareBtn.classList.remove('hidden');

                };
            }
        };
    }
  
    // Share using Web Share API
    async function sharePoster() {
      canvas.toBlob(async (blob) => {
        const file = new File([blob], 'poster.png', { type: 'image/png' });
        if (navigator.share) {
          await navigator.share({
            title: 'Check out my poster!',
            files: [file],
          });
        } else {
          console.log('Web Share API is not supported in this browser.');
          downloadPoster();
        }
      });
    }

    // Download poster
    function downloadPoster() {
      const link = document.createElement('a');
      link.download = 'poster.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    }

    function restart(){
      canvas.classList.add('fixed');
      imageSelect.classList.remove('hidden');
      shareBtn.classList.add('hidden');
    }

    function resizeToMax200(width, height) {
      const maxSize = 200;
      let newWidth, newHeight;

      if (width < height) {
        // Width is the larger side
        newWidth = maxSize;
        newHeight = (height / width) * maxSize;
      } else {
        // Height is the larger side
        newHeight = maxSize;
        newWidth = (width / height) * maxSize;
      }

      return { width: newWidth, height: newHeight };
    }

    function isCaptureSupported() {
      // Check if the browser can access media devices
      return !!navigator.mediaDevices && /Android|iPhone|iPad/i.test(navigator.userAgent);
    }

    onMount(() => {
      console.log(navigator.mediaDevices);
      if (!isCaptureSupported()) {
        selfieBooth.classList.add('hidden');
      }
    });

  </script>
  
  <div class="bg-white rounded-md shadow-md p-5 max-w-xl mx-auto">
   
    <div class="text-center font-bold mb-5">Generate Your Poster</div>
    
    <div bind:this="{imageSelect}" class="flex items-center justify-center gap-5 mb-5">
        <label for="upload-file-1" bind:this="{selfieBooth}" class="border border-2 border-dashed p-5 rounded-lg flex flex-col items-center">
            <img src="img/selfie.svg" alt="Take Selfie" class="mb-2 h-40"/>
            <div class="text-center">Take Selfie</div>
        </label>

        <label for="upload-file-2" class="border border-2 border-dashed p-5 rounded-lg flex flex-col items-center">
          <img src="img/file-upload.svg" alt="Take Selfie" class="mb-2 h-40"/>
          <div class="text-center">Upload Image</div>
      </label>
    </div>

    <canvas bind:this="{canvas}" width="1080" height="1080" class="w-full fixed -z-10 mb-5"></canvas>

    <div class="text-center hidden flex items-center gap-3 flex-wrap justify-center" bind:this="{shareBtn}">
      <button on:click="{sharePoster}"  class="px-5 py-2 bg-[#444b53] text-white rounded-lg shadow-md flex gap-2">
        <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="w-5 icon icon-tabler icons-tabler-outline icon-tabler-share"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M6 12m-3 0a3 3 0 1 0 6 0a3 3 0 1 0 -6 0" /><path d="M18 6m-3 0a3 3 0 1 0 6 0a3 3 0 1 0 -6 0" /><path d="M18 18m-3 0a3 3 0 1 0 6 0a3 3 0 1 0 -6 0" /><path d="M8.7 10.7l6.6 -3.4" /><path d="M8.7 13.3l6.6 3.4" /></svg>
        <span>Share</span>
      </button>
      <button on:click="{downloadPoster}"  class="px-5 py-2 bg-[#444b53] text-white rounded-lg shadow-md flex gap-2">
        <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="w-5 icon icon-tabler icons-tabler-outline icon-tabler-download"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M4 17v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2 -2v-2" /><path d="M7 11l5 5l5 -5" /><path d="M12 4l0 12" /></svg>
        <span>Download</span>
      </button>
      <button on:click="{restart}"  class="px-5 py-2 bg-[#444b53] text-white rounded-lg shadow-md flex gap-2">
        <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="w-5 icon icon-tabler icons-tabler-outline icon-tabler-arrow-back-up"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M9 14l-4 -4l4 -4" /><path d="M5 10h11a4 4 0 1 1 0 8h-1" /></svg>
        Make Another
      </button>
    </div>

    <input id="upload-file-1" class="hidden" type="file" accept="image/*" capture="user" on:change="{handleImageUpload}" />
    <input id="upload-file-2" class="hidden" type="file" accept="image/*" on:change="{handleImageUpload}" />
    
  </div>


  