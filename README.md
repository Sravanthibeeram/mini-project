// html code

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="style.css">
        
</head>
<body>
    <h1>Steganography Project</h1>
    <hr width="100%">
    <h2>Enter text to hide in image  </h2>
    <!-- <input type="text" placeholder="enter text" class="content"> -->
    <textarea rows="4" cols="40" class="content" placeholder="Enter text to hide" style="resize: none;"></textarea>
    
    <h2>
        <label for="files" class="labelFiles">Choose File : </label>
        <!-- <h3 class="alert">pls upload jpg/png only</h3> -->
        <input type="file" id="files" accept="image/*"  placeholder="hell"/>
    </h2>
    <!-- <img src="" class="hidden originalImage"> -->
    <div class="options">
        <button class="encrypt"><h2>Hide the message</h2></button>
    <button class="extract"><h2>Extract message</h2></button>

    </div>

    <!-- part for encryption -->
    <div class="result">
        <div class="cols">
            <h2>Before Modification</h2>
            <img src="" class="originalImage">
        </div>
        <div class="cols">
            <h2>After Modification</h2>
            <canvas id="imageCanvas" ></canvas>
        </div>
    </div>
    <button class="download hidden" href="" download><h2>DOWNLOAD</h2></button>
    
    <!-- part for data extraction -->
    <div class="data-extracted hidden">
        <h2>Selected image</h2>
        <canvas id="imageCanvas2" ></canvas>
        <h2>Hidden data</h2>
        <div class="notice" style="color:red; display:none">
            no hidden data / (make sure you are using the downloded file for extraction)
        </div>
        <textarea rows="4" cols="40" class="extracted-item" placeholder="hidden data" style="resize: none;"></textarea>
    </div>

    

    <script src="main.js"></script>
    
</body>
</html>

//java code

let file_uploded = 0;
let imageCanvas = document.querySelector('#imageCanvas');

//button used for file selection
const fileInput = document.querySelector("#files");

//this function displays the image as soon as image is selected
// let selectedFile = null;
console.log("hello world");
fileInput.addEventListener('change',(event)=>{
    file_uploded = 1;  //file uploaded
    let selectedFile = event.target.files[0];
  //created url to present selected image
    const url = URL.createObjectURL(selectedFile); 
    document.querySelector('.originalImage').src = url;
    // document.querySelector('.for-extraction').src = url;
    document.querySelector('.originalImage').classList.remove('hidden');
});

//encrypt is the button that will fire function for data hiding
const encrypt = document.querySelector('.encrypt');

encrypt.addEventListener('click',()=>{
    //only proceed if image has been selected
    if(file_uploded===1) {
    //main function start
        console.log('ready');
        createImageCanvas();
        document.querySelector('.download').style.display = "block";
        // startHiding();
        document.querySelector('.result').style.display="flex";
        document.querySelector('.data-extracted').style.display = "none";
    }
    else alert("select image file");

});
        

function createImageCanvas(){
    // let selectedFile = fileInput.target.files;
    // imageCanvas.height = 
    let img = new Image();
    img.src = URL.createObjectURL(fileInput.files[0]);
    img.onload = function(){
        imageCanvas.width = img.width;
        imageCanvas.height = img.height;
        const ctx = imageCanvas.getContext("2d");
        ctx.drawImage(img, 0, 0);
        // console.log(img.height+" "+img.width);
        const imageData = ctx.getImageData(0,0,imageCanvas.width,imageCanvas.height);
        const pixelData = imageData.data;       //array containg pixel info
        // console.log("original pixel data " + pixelData);
        let message = document.querySelector('.content').value;
        message = ("Valid\n"+message);
        // console.log("message to be hidden" + message);
        
        const arr = [];

        //converting message to binary array
        for(let i=0;i<message.length;i++) {
            let x = message[i].charCodeAt(0);
            let  a=128;
            while(a>0){
                if(((a&x) === a)) {
                    
                    arr.push(1);
                }
                else arr.push(0);
                a>>=1;
                
            }   
        }
        
        //to make target pixels 0;
        
        // console.log("binary array of message " + arr);
        for (let i = 0; ((i < pixelData.length)); i += 1) {
            pixelData[i] &=252 ; 
        }
        
        //* *Reminder 
        // Write a case when text is too big

        //this part is to add data in last 2 bits

        let ctr=0;
        for(let i=0;i<arr.length;i+=2){
            pixelData[ctr] += (2*arr[i] + arr[i+1]);
            ctr++;
        }
        
        ctx.putImageData(imageData, 0, 0);
        // console.log("has it changed " + imageData.data);
          
    }

    document.querySelector('.download').addEventListener('click',()=>{
        imageCanvas.toBlob(function(blob) {
            // Create a new anchor element with a download attribute
            const downloadLink = document.createElement('a');
            downloadLink.download = 'myCanvasImage.png';
        
            // Create a URL for the blob
            const url = URL.createObjectURL(blob);
        
            // Set the download link's href to the URL of the blob
            downloadLink.href = url;
        
            // Trigger a click on the download link to download the image
            downloadLink.click();
        
            // Release the URL object
            URL.revokeObjectURL(url);
          }, 'image/png');
    }); 

    //for Download button
    const download = document.querySelector('.download');
    download.href = img.src;
    
    
}


// logic for extract button
document.querySelector('.extract').addEventListener('click',()=>{
    if(file_uploded===1) {
        //main function start
            // document.querySelector('.extracted').classList.remove('hidden');

            document.querySelector('.result').style.display = "none";
            document.querySelector('.download').style.display = "none"; 
            document.querySelector('.data-extracted').style.display = "flex";

            let imageCanvas2 = document.querySelector('#imageCanvas2');
            let img = new Image();
            img.src = URL.createObjectURL(fileInput.files[0]);
            img.onload = function(){
                imageCanvas2.width = img.width;
                imageCanvas2.height = img.height;
                let ctxt= imageCanvas2.getContext("2d");
                ctxt.drawImage(img, 0, 0);
                let imageData = ctxt.getImageData(0,0,imageCanvas2.width,imageCanvas2.height);
                const pixelData = imageData.data;
                // console.log(pixelData);
                const data_wali_array = [];
                for(let i=0;i<pixelData.length;i++){
                    data_wali_array.push((((pixelData[i]&2)!=0)?1:0));
                    data_wali_array.push(pixelData[i]&1);
                }
                console.log(data_wali_array);
                let ans="";
                let x = 0;
                let flag=1;
                let count=0;
                for(let i=0;i<data_wali_array.length;i++) {
                    let a=1;
                    console.log("hemlo");
                    if(data_wali_array[i]==1){
                        a<<=(7-i%8);
                        x+=a;
                        // continue;
                    }
                    if(i%8===7){
                        count++;
                        if(String.fromCharCode(x)==='\0'){
                            break;
                        }
                        x%=128;
                        // console.log('y');
                        ans+=String.fromCharCode(x);
                        x=0;
                    }
                    if(ans.length===5 &&  ans!=="Valid") {
                        document.querySelector('.extracted-item').value = "";
                        // console.log("not encoded" + ans); 
                        // alert("no data hidden");
                        document.querySelector('.notice').style.display = "block";
                        console.log("not encoded" + ans); 
                        flag=0;
                        break;
                    }
                    // if(count>10000) break;
                }
                if(flag) {
                    document.querySelector('.notice').style.display = "none";
                    document.querySelector('.extracted-item').value = ans.substring(6);
                    console.log(ans);
                }
                console.log("decoded");
            }
            
        }
        else alert("select image file");
});

// css code

body {
    background-color:  #1D1F21;;
    color: darkgray;
    font-family: Arial, Helvetica, sans-serif;
    align-items: center;
    justify-content: center;
    display: flex;
    flex-direction: column;
}
.hidden {
    display: none;
}
.originalImage {
    height: 300px;
    width: 300px;
    /* margin-top: 20px;
    margin-bottom: 20px; */
    margin-left: 20px;
    margin-right: 20px;
    border: #bac4ce00 solid;
    background-color: rgba(127, 255, 212, 0);
}
.imageCanvas {
    background-color: aliceblue;
}
canvas{
    width: 300px;
    height: 300px;
}
.result {
    display: none;
    flex-direction: row;
    width: 100%;
    align-items: center;
    justify-content: center;
}
.cols{
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin: 20px;
}
@media screen and (max-width: 650px) {
    .result{
        /* display: flex; */
        flex-direction: column;
    }
}
/* img{
    width: 300px;
    height: 300px;
} */
.options{
    display: flex;
    justify-content: center;
    align-items: center; 
}
button{
    margin:10px;
}
img{
    height: 300px;
    width: 300px;
}
.data-extracted {
    /* display: flex; */
    /* display: flex; */
    display: none;
    flex-direction: column;
    align-items: center;
    justify-content: space-between  ;
    
}
