# interactive


## 프로젝트 공정

<br>

#### 1. HTML, CSS 작업

<br>

#### 2. 각 Section별로 어떤식으로 구현할지 (Scroll line으로 전체적인 액션)
> ##### #scroll-section-0
> ##### 비디오 재생 중간에 텍스트 애니메이션 
> ##### #scroll-section-1
> ##### 보통 scroll 영역
> ##### #scroll-section-2
> ##### 비디오 재생 중간에 텍스트 애니메이션
> ##### #scroll-section-3
> ##### 처음 보통 scroll -> 콘텐츠가 커지면서 다른이미지로 블렌딩 -> 다시 작아지면서 -> 보통 scroll  

<br>

---

<br>

# __Point__

### - CSS
> #### 헤더 정렬 display: flex

<br>

### - 텍스트 스크립트 처리방식
> #### body에 show-scene0 클래스를 붙여 sticky-elem(텍스트)을 구간에 맞게 보여지게 한다.
> #### #show-scene-0 #scroll-section-0 .sticky-elem,
> #### #show-scene-1 #scroll-section-1 .sticky-elem,
> #### #show-scene-2 #scroll-section-2 .sticky-elem,
> #### #show-scene-3 #scroll-section-3 .sticky-elem {
> ####    display: block;
> #### } 

<br>

### - 각 섹션의 높이 설정
> #### 함수를 만들고 호출 되게 한다.
> #### (() => {}) ()
> #### 각 scene에 대한 정보를 객체에 담는다.
> #### const sceneInfo = []
> #### sceneInfo안에 type을 정하고, 브라우저 높이의 5배로 scrollHeight 세팅하기 위해(heightNum, scrollHeight)설정, objs: {}에 HTML 객체 담아 둔다.
```
 (() => {
    const sceneInfo = [ // 각 scene에 대한 정보를 객체에 담는다.
        { 
            // 첫번째 객체 0
            type: 'sticky',
            heightNum: 5,
            scrollHeight: 0, 
            objs: {
                container: document.querySelector('#scroll-section-0')
            }
        ]
   }
```
> ####  각 스크롤 섹션의 높이 세팅
```     
    function setLayout() {
        for (let i = 0; i < sceneInfo.length; i++) {
            // 섹션 높이 설정
            sceneInfo[i].scrollHeight = sceneInfo[i].heightNum * window.innerHeight;
            // 섹션 선택
            sceneInfo[i].objs.container.style.height = `${sceneInfo[i].scrollHeight}px`;      
        }
    }
    window.addEventListener('resize', setLayout); // 윈도우 창의 사이즈가 변할때 resize
    setLayout();
```

<br>

### - 현재 스크롤 한 위치 변수지정(yOffset)
```
console.log(window.pageYOffset);
```

<br>

### - window.addEventListener('scroll', () => {}) 추가

<br>

### - scrollLoop() 함수 추가 => 현재 눈에보이는 화면 캐치
> #### 예를들어, 2번 섹션의 상단과 브라우저창의 상단이 겹쳐질때 시작지점으로 본다. 
> #### 값은 전체 스크롤 된 값(yOffset)으로 할 수 없으니 이전에 있는 0번 섹션과 1번 섹션의 스크롤 값(높이)을 기준으로 잡는다.
> #### 이전의 0,1 섹션의 높이보다 크면은 2섹션이 시작되었다고 볼 수 있다.

<br>

### - 현재 활성시킬 씬 결정하기

<br>

### - let prevScrollHeight 변수 추가 => 현재 스크롤 위치(yOffset)보다 이전에 위치한 스크롤 섹션들의 스크롤 높이값의 합

<br>

### - let currentScene 변수 추가 => 현재 활성화된(눈 앞에 보고있는) 씬(scroll-section)
> #### prevScrollHeight 값과 currentScene값을 비교하여 currentScene 몇번인지 판별

<br>

### - scrollLoop() 함수 세팅
> #### 메뉴의 높이 반영 또는 공간을 차지하지 않는(fixed, absolute) CSS변경

<br>
<br>

## __스크롤 애니메이션 구현1__
### body에 id부여
```
document.body.setAttribute('id', `show-scene-${currentScene}
```

<br>

### 처음에 로드 되거나 새로 고침했을 경우 body에 id 부여가 정확하지가 않다.
### setLayout()초기화 관련이라서 setLayout()에서 잡아준다.
### setLayout() 변경
```
function setLayout() {
        for (let i = 0; i < sceneInfo.length; i++) {
            // 섹션 높이 설정
            sceneInfo[i].scrollHeight = sceneInfo[i].heightNum * window.innerHeight;
            // 섹션 선택
            sceneInfo[i].objs.container.style.height = `${sceneInfo[i].scrollHeight}px`;  // `` template 문자열 -> ${변수/값}
             
        }

        yOffset = window.pageYOffset;
        let totalScrollHeight = 0;
        for (let i = 0; i < sceneInfo.length; i++) {
            totalScrollHeight += sceneInfo[i].scrollHeight;
            if (totalScrollHeight >= yOffset) {
                currentScene = i;
                break;
            }
        } 
        document.body.setAttribute('id', `show-scene-${currentScene}`);
    }
```
> #### DOMContentLoaded DOM로드가 끝나면 바로 실행, 실행 시점이 좀 더 빠르다.
> #### load 이미지등 모든것이 준비되면 실행
```
// window.addEventListener('DOMContentLoaded', setLayout); 
window.addEventListener('load', setLayout);
window.addEventListener('resize', setLayout);
```

<br>

### 첫번째 객체에 DOM 객체를 넣는다.
> #### 클래스 main-message에 a를 추가, values에 messageA_opacity 추가, 현재 스크롤 량 (각 섹션마다)
> #### (currentYOffset는 2섹션일 경우 0, 1 섹션을 빼면 2섹션에서 스크롤 된 양을 알수 있다.)
```
function playAnimation () {
    const currentYOffset = yOffset - prevScrollHeight;
    console.log(currentScene, currentYOffset)
}
```
> #### 해당 섹션에서 현재 씬(스크롤 섹션)에서 스크롤된 범위를 비율로 구하기
```
function calcValues(values, currentYOffset) {
        let rv;
        // 현재 씬(스크롤 섹션)에서 스크롤된 범위를 비율로 구하기(0~1)
        let scrollRatio = currentYOffset / sceneInfo[currentScene].scrollHeight;
        return scrollRatio;
    }
```
> #### 스크롤된 범위를 비율로 구한 값에 범위 지정
> #### 범위가 200~900일 경우 (900 - 200) + 200을 하면 200에서 시작하여 900에서 끝난다.
> #### 보기 편하게 parseInt 후에 삭제
```
function calcValues(values, currentYOffset) {
        let rv;
        let scrollRatio = currentYOffset / sceneInfo[currentScene].scrollHeight; // 현재 씬(스크롤 섹션)에서 스크롤된 범위를 비율로 구하기(0~1)
        
        rv = parseInt(scrollRatio * (values[1]-values[0]) + values[0]); // parseInt = 소수점 없애기, 300 전체 범위가 300일 경우 300으로 범위설정

        return rv;
    }
```
### 씬이 끝날때 같이 끝날 수 있도록 구현
### scrollLoop()함수에 playAnimation()추가, playAnimation()함수 추가
```
function playAnimation () {
        switch (currentScene) {
            case 0:
                // console.log('0 play');
                break;
            case 1:
                // console.log('1 play');
                break;
            case 2:
                // console.log('2 play');
                break;
            case 3:
                // console.log('3 play');
                break;
        }
    }
```
### 스크롤 된 범위(0~1)을 opacity값으로 준다.
### .main-message의 스타일을 opacity: 0;
```
switch (currentScene) {
            case 0:
                // console.log('0 play');
                // 텍스트 opacity
                let messageA_opacity_in = calcValues(values.messageA_opacity, currentYOffset);
                objs.messageA.style.opacity = messageA_opacity_in;
                break;
```
### 그러나 텍스트는 마지막에 opacity값이 1이 된다 => 키프레임을 넣어주어야 한다.
### 현재 등장만할뿐, 사라지지는 않는다. (일부분에서만 작동해야 한다.)
### 씬이 바뀌는 순간 값이 한번 이상하게 나온다.(스크롤 양등..) => 씬을 바꿔주는 scrollLoop()에서 수정
> #### 변수를 만들어서 씬이 바뀔때 변수가 들어간다 그다음 턴에서 false 처리
```
let enterNewScene = false; // 새로운 씬이 시작된 순간 true
```
> #### playAnimation가 값을 실행한다. => 아주 잠깐 if (enterNewScene) return 되면서 한번 걸러진다.
```
function scrollLoop() {
        enterNewScene = false;
        prevScrollHeight = 0;
        for (let i = 0; i < currentScene; i++) {
            prevScrollHeight += sceneInfo[i].scrollHeight;
        }

        if (yOffset > prevScrollHeight + sceneInfo[currentScene].scrollHeight) {
            enterNewScene = true;
            currentScene ++;
            document.body.setAttribute('id', `show-scene-${currentScene}`);
        }

        if (yOffset < prevScrollHeight) {
            if (currentScene === 0) return; 
            enterNewScene = true;
            currentScene --;
            document.body.setAttribute('id', `show-scene-${currentScene}`);
        } 

        if (enterNewScene) return;
        
        playAnimation();
    }
```
> #### sceneInfo의 valuse에 start, end를 넣어 시작과 끝을 준다.(타이밍)
> #### messageA는 10%~20%, messageB는 20%~40%
```
values: {
    messageA_opacity: [0, 1, { strat: 0.1, end: 0.2 }],
    messageB_opacity: [0, 1, { strat: 0.2, end: 0.4 }],
}
```
> #### 1섹션이 4000px일 경우 10~20%구간을 설정할 경우 20% - 10%는 해당 구간 / 시작은 10% * 해당 구간 / 끝은 20% * 해당 구간
```
function calcValues(values, currentYOffset) {
        let rv;
        const scrollHeight = sceneInfo[currentScene].scrollHeight;
        const scrollRatio = currentYOffset / scrollHeight; // 현재 씬(스크롤 섹션)에서 스크롤된 범위를 비율로 구하기(0~1)
        
        if (values.length === 3) {
            // strat ~ end 사이에 애니메이션 실행
            const partScrollStart = values[2].start * scrollHeight;
			const partScrollEnd = values[2].end * scrollHeight;
			const partScrollHeight = partScrollEnd - partScrollStart;
            
            if (currentYOffset >= partScrollStart && currentYOffset <= partScrollEnd) {
				rv = (currentYOffset - partScrollStart) / partScrollHeight * (values[1] - values[0]) + values[0];
			} else if (currentYOffset < partScrollStart) {
				rv = values[0];
			} else if (currentYOffset > partScrollEnd) {
				rv = values[1];
			}
        } else {
            rv = scrollRatio * (values[1]-values[0]) + values[0]; // parseInt = 소수점 없애기, 300 전체 범위가 300일 경우 300으로 범위설정
        }

        return rv;
    }
```
> #### .main-message CSS top: 35vh
> #### messageA_opacity_in 변경 후 messageA_opacity_out 설정(사라짐) 그런데 if문에서 messageA_opacity_in은 구간 이후 1이 유지되어야 하는데 messageA_opacity_out의 값이 있기때문에 구간별로 나누어서 설정한다. 
>> #### in범위 / out범위 => messageA_opacity_in 0.1 ~ 0.2 / messageA_opacity_out 0.25 ~ 0.3, 0.2와 0.25 사이의 값 0.22로 in과 out을 나누어 준다. 
>> #### playAnimation()에 변수 추가 후 case 0에서 구간 나눠준다.
```
function playAnimation () {
        const objs = sceneInfo[currentScene].objs;
        const values = sceneInfo[currentScene].values;
        const currentYOffset = yOffset - prevScrollHeight;
        const scrollHeight = sceneInfo[currentScene].scrollHeight; // 현재 씬의 scrollHeight
        const scrollRatio = (yOffset - prevScrollHeight) / scrollHeight; // in/out 구간 나누기위해 현재 씬에서 얼마나 스크롤 했는지의 비율

        // console.log(currentScene)

        switch (currentScene) {
            case 0:
                // console.log('0 play');
                // 텍스트 opacity
                const messageA_opacity_in = calcValues(values.messageA_opacity_in, currentYOffset);
                const messageA_opacity_out = calcValues(values.messageA_opacity_out, currentYOffset);
                
                if (scrollRatio <= 0.22) {
                    // in
                    objs.messageA.style.opacity = messageA_opacity_in;
                } else {
                    // out
                    objs.messageA.style.opacity = messageA_opacity_out;
                }
                
                break;
```

<br>

> #### transform 추가 objs > values > messageA_translateY_in 추가 
```
values: {
    messageA_opacity_in: [0, 1, { start: 0.1, end: 0.2 }],
    // messageB_opacity_in: [0, 1, { start: 0.3, end: 0.4 }],
    messageA_translateY_in: [20, 0, { start: 0.1, end: 0.2 }], // 20에서 0%, 타이밍
    
    messageA_opacity_out: [1, 0, { start: 0.25, end: 0.3 }],
    messageA_translateY_out: [0, -20, { start: 0.25, end: 0.3 }],
}




switch (currentScene) {
    case 0:
        // console.log('0 play');
        // 텍스트 opacity
        const messageA_opacity_in = calcValues(values.messageA_opacity_in, currentYOffset);
        const messageA_opacity_out = calcValues(values.messageA_opacity_out, currentYOffset);
        const messageA_translateY_in = calcValues(values.messageA_translateY_in, currentYOffset);
        const messageA_translateY_out = calcValues(values.messageA_translateY_out, currentYOffset);
        
        if (scrollRatio <= 0.22) {
            // in
            objs.messageA.style.opacity = messageA_opacity_in;
            objs.messageA.style.transform = `translateY(${messageA_translateY_in}%)`;
        } else {
            // out
            objs.messageA.style.opacity = messageA_opacity_out;
            objs.messageA.style.transform = `translateY(${messageA_translateY_out}%)`;
        }
        
        break;
```

<br>

### 자잘한 수정사항 처리
> #### type이 normal인 속성 작업
```
function setLayout() {
    for (let i = 0; i < sceneInfo.length; i++) {
        if (sceneInfo[i].type === 'sticky') {
            // 섹션 높이 설정
            sceneInfo[i].scrollHeight = sceneInfo[i].heightNum * window.innerHeight;
        } else if (sceneInfo[i].type === 'normal') {
            sceneInfo[i].scrollHeight = sceneInfo[i].objs.container.offsetHeight;
        }
        // 섹션 선택
        sceneInfo[i].objs.container.style.height = `${sceneInfo[i].scrollHeight}px`;  // `` template 문자열 -> ${변수/값}
    }
```
> #### 변수처리 수정
```
const scrollRatio = currentYOffset / scrollHeight; // in/out 구간 나누기위해 현재 씬에서 얼마나 스크롤 했는지의 비율
```
```
switch (currentScene) {
    case 0:
        // console.log('0 play');
        // 텍스트 opacity
        if (scrollRatio <= 0.22) {
            // in
            objs.messageA.style.opacity = calcValues(values.messageA_opacity_in, currentYOffset);
            objs.messageA.style.transform = `translateY(${calcValues(values.messageA_translateY_in, currentYOffset)}%)`;
        } else {
            // out
            objs.messageA.style.opacity = calcValues(values.messageA_opacity_out, currentYOffset);
            objs.messageA.style.transform = `translateY(${calcValues(values.messageA_translateY_out, currentYOffset)}%)`;
        }
```
> #### HTML 클래스 a,b,c 추가
```
<div class="sticky-elem main-message a">
```
> #### CSS 추가
```
#scroll-section-2 .b {
    top: 10%;
    left: 40%;
}
#scroll-section-2 .c {
    top: 15%;
    left: 45%;
}
```
> #### main-add.js로 추가

<br>

### 비디오로 고해상도로할 경우 많이 버벅인다.
### 이미지를 사용할 경우 고해상도에 부드럽다.(비디오를 프레임 하나하나 다 뽑았다.) => 16초짜리 초당 60프레임으로 만듬(16*60 = 960 장)
### 구글에 동영상 프레임 추출 검색 => 이미지 압축 한번 해준다.
### 비디오 인터랙션 적용
> #### 캔버스 추가
```
<div class="sticky-elem sticky-elem-canvas">
    <canvas id="video-canvas-0" width="1920" height="1080"></canvas>
</div>
```
> #### .global-nav / .local-nav에 z-index 추가
> #### obj, values 설정
```
canvas: document.querySelector('#video-canvas-0'),
context: document.querySelector('#video-canvas-0').getContext('2d'),
videoImages: []


values: {
    videoImagesCount: 300, // 이미지 갯수
    imageSequenece: [0, 299], // 이미지 순서
```
> #### 비디오 이미지 객체를 생성하여 videoImages배열에 담는다.
```
// 캔버스에 처리할 이미지들을 세팅
function setCanvasImages() {
    let imgElem;
    for (let i = 0; i < sceneInfo[0].values.videoImagesCount; i++) {
        imgElem = new Image();
        imgElem.src = `./assets/img/001/IMG_${6726 + i}.JPG`; // index.html에서 호출하여 index 기준   
        sceneInfo[0].objs.videoImages.push(imgElem);
    }
}
setCanvasImages();
```
> #### 스크롤에 따라서 이미지를 그린다. playAnimation()함수에서
>> ##### 텍스트와 다르게 초기에 나타나 마지막에 없어지면 된다. scrollRatio가 필요 없다.
```
switch (currentScene) {
    case 0:
        let sequence = Math.round(calcValues(values.imageSequenece, currentYOffset)); // 이미지 순서
        objs.context.drawImage(objs.videoImages[sequence], 0, 0) // 이미지, x, y
```
> #### 캔버스 크기 조정 => setLayout() 에서 맞추면 적절하다.(로드 후, 리사이즈 후)
>> ##### CSS로 조정 시 성능도 좋고 쉽다(transform:scale), 캔버스 자체 조정 시 픽셀을 줄인다.
>> ##### height를 맞춘다. 16:9 => height 고정시키고 가로 정렬
```
const heightRatio = window.innerHeight / 1080; // 윈도우 창의 비율
sceneInfo[0].objs.canvas.style.transform = `scale(${heightRatio})`;
```
>> ##### CSS로 transform:scale로 잡혀있는 섹션 중앙 정렬하기
>> ##### 이미 자바스크립트로 transform 요소를 건드리고있기때문에 스크립트로 같이 넣어준다.
```
sceneInfo[0].objs.canvas.style.transform = `translate3d(-50%, -50%, 0) scale(${heightRatio})`;
```
>> ##### canvas 배경 #ccc 삭제
> #### 새로고침하거나 로드 되었을 시 안그려진다. => 스크롤 시 그려지게 만들었다.
```
window.addEventListener('load', setLayout)
after
window.addEventListener('load', () => {
    setLayout();
    sceneInfo[0].objs.context.drawImage(sceneInfo[0].objs.videoImages[0], 0, 0);
}); 
```
> #### AirMug Pro 텍스트 스타일 수정
```
#scroll-section-0 h1 {
    position: relative;
    top: -10vh;
    z-index: 5;
}
```
> #### 다음 섹션으로 넘어갈때 확 없어진다. => 똑같이 캔버스의 opacity값을 조정해준다.
>> ##### sceneInfo values 값에 추가
```
canvas_opacity: [1, 0, { start: 0.9, end: 1 }], // 캔버스 
```
>> ##### 캔버스 opacity는 한번만 실행하면 된다.
```
objs.canvas.style.opacity = calcValues(values.canvas_opacity, currentYOffset); 
```

<br>

### 두번째 스크롤 비디오 처리
> #### HTML canvas 추가
```
<!-- section 03  -->
<section class="scroll-section" id="scroll-section-2">
    <div class="sticky-elem sticky-elem-canvas">
        <canvas id="video-canvas-1" width="1920" height="1080"></canvas>
    </div>
```
> #### 객체 2번에 위와같이 objs, values에 canvas 추가
```
objs
canvas: document.querySelector('#video-canvas-1'),
context: document.querySelector('#video-canvas-1').getContext('2d'),
videoImages: []

values
videoImagesCount: 300,
imageSequenece: [0, 299],
canvas_opacity: [1, 0, { start: 0.9, end: 1 }],
```
> #### setLayout()에 캠버스 사이즈 조절
```
sceneInfo[2].objs.canvas.style.transform = `translate3d(-50%, -50%, 0) scale(${heightRatio})`;
```
> #### playAnimation()에 case2에 변수 sequence, context 추가
```
let sequence2 = Math.round(calcValues(values.imageSequenece, currentYOffset)); // 이미지 순서
objs.context.drawImage(objs.videoImages[sequence2], 0, 0); // 이미지, x, y
```
> #### setCanvasImages()함수에 이미지를 추가
```
let imgElem2;
for (let i = 0; i < sceneInfo[2].values.videoImagesCount; i++) {
    imgElem2 = new Image();
    imgElem2.src = `./assets/img/002/IMG_${7027 + i}.JPG`;
    sceneInfo[2].objs.videoImages.push(imgElem2);
}
```
> #### 등장, 아웃 시에 투명도 적용 => case2에 scrollRatio 적용  
```
if (scrollRatio <= 0.5) {
    // in
    objs.canvas.style.opacity = calcValues(values.canvas_opacity_in, currentYOffset);
} else {
    // out
    objs.canvas.style.opacity = calcValues(values.canvas_opacity_out, currentYOffset);
}
```

<br>

### 마지막 씬
> #### HTML canvas 추가
```
<canvas class="image-blend-canvas" width="1920" height="1080"></canvas>
```
> #### 객체3에 캔버스 선택, context 추가
```
container: document.querySelector('#scroll-section-3'),
canvasCaption: document.querySelector('.canvas-caption'),
canvas: document.querySelector('.image-blend-canvas'),
context: document.querySelector('.image-blend-canvas').getContext('2d'),
```
> #### 화면 크기나 비율에 따라 유동적으로 변화가 필요해 setLayout()가 아니라 playAnimation()에서 스크롤하면서 적용 시킨다. => 가로/세로 모두 꽉 차게 하기 위해 여기서 세팅(계산 필요)
```
case 3:
    // 가로/세로 모두 꽉 차게 하기 위해 여기서 세팅(계산 필요)
    const widthRatio = window.innerWidth / objs.canvas.width; // width 비율
    const heightRatio = window.innerHeight / objs.canvas.height; // height 비율
    let canvasScaleRatio;
    
    if (widthRatio <= heightRatio) {
        // 캔버스보다 브라우저 창이 홀쭉한 경우
        canvasScaleRatio = heightRatio;
    } else {
        // 캔버스보다 브라우저 창이 납작한 경우
        canvasScaleRatio = widthRatio;
    }

    objs.canvas.style.transform = `scale(${canvasScaleRatio})`
    
```
> #### setCanvasImages()에 캔버스에 그릴 이미지 담기 
```
let imgElem3;
for (let i = 0; i < sceneInfo[3].objs.imagesPath.length; i++) {
    imgElem3 = new Image();
    imgElem3.src = sceneInfo[3].objs.imagesPath[i];
    sceneInfo[3].objs.images.push(imgElem3);
}
```
> #### 이미지 그리기
```
case 3:

objs.canvas.style.transform = `scale(${canvasScaleRatio})`
objs.context.drawImage(objs.images[0], 0, 0);
```
> #### 하얀 박스를 그려주고 스크롤 시 박스가 이동한다고 생각(캐릭터는 그대로)
```
{ 
    // 3
    type: 'sticky',
    heightNum: 5, 
    scrollHeight: 0, 
    objs: { 
        container: document.querySelector('#scroll-section-3'),
        canvasCaption: document.querySelector('.canvas-caption'),
        canvas: document.querySelector('.image-blend-canvas'),
        context: document.querySelector('.image-blend-canvas').getContext('2d'),
        imagesPath: [
            './assets/img/blend-image-1.jpg',
            './assets/img/blend-image-2.jpg'
        ],
        images: []
    },
    values: {
        rect1X: [0, 0, { start: 0, end: 0 }],
        rect2X: [0, 0, { start: 0, end: 0 }],
    }
}
```
> #### 사이즈, 위치 구하기
```
case 3:
    // console.log('3 play');
    // 가로/세로 모두 꽉 차게 하기 위해 여기서 세팅(계산 필요)
    const widthRatio = window.innerWidth / objs.canvas.width; // width 비율
    const heightRatio = window.innerHeight / objs.canvas.height; // height 비율
    let canvasScaleRatio;
    
    if (widthRatio <= heightRatio) {
        // 캔버스보다 브라우저 창이 홀쭉한 경우
        canvasScaleRatio = heightRatio;
    } else {
        // 캔버스보다 브라우저 창이 납작한 경우
        canvasScaleRatio = widthRatio;
    }

    objs.canvas.style.transform = `scale(${canvasScaleRatio})`
    objs.context.drawImage(objs.images[0], 0, 0);

    // 캔버스 사이즈에 맞춰 가정한 innerWidth와 innerHeight
    const recalculatedInnerWidth = window.innerWidth / canvasScaleRatio;
    const recalculatedInnerHeight = window.innerHeight / canvasScaleRatio;

    const whiteRectWidth = recalculatedInnerWidth * 0.15;
    values.rect1X[0] = (objs.canvas.width - recalculatedInnerWidth) / 2;
    values.rect1X[1] = values.rect1X[0] - whiteRectWidth;
    values.rect2X[0] = values.rect1X[0] + recalculatedInnerWidth - whiteRectWidth;
    values.rect2X[1] = values.rect2X[0] + whiteRectWidth;
    
    // // 좌우 흰색 박스 그리기
    objs.context.fillRect(values.rect1X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight);
    objs.context.fillRect(values.rect2X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight);
    // break;
```
> #### objs > values에 박스 시작점 끝점 구하기 (스크롤 할때 값을 계산)
>> #### 애니메이션이 시작되는 지점은 0, 애니메이션이 종료되는 시점은 시작지점에서 캔버스 사이의 값을 구해 종료 시점으로 한다.
>> #### getBoundingClientRect() 메서드(화면상에 있는 오브젝트의 크기와 위치를 가져올 수 있다.) 이용
```
console.log(objs.canvas.getBoundingClientRect())
```
>> #### values 값을 받을 변수 추가 => 처음에 값을 받은 후 처음값이 변하지 않게 한다. (기준점으로 사용)
```
values: {
    rect1X: [0, 0, { start: 0, end: 0 }],
    rect2X: [0, 0, { start: 0, end: 0 }],
    rectStartY: 0,
}


if (!values.rectStartY) { 
    values.rectStartY = objs.canvas.getBoundingClientRect().top;
    values.rect1X[2].end = values.rectStartY / scrollHeight;
    values.rect2X[2].end = values.rectStartY / scrollHeight;
}
```
>> #### 그런데, 스크롤 속도에 따라 값이 바뀐다. => offsetTop()을 사용
```
// 캔버스 사이즈에 맞춰 가정한 innerWidth와 innerHeight
const recalculatedInnerWidth = window.innerHeight / canvasScaleRatio;
const recalculatedInnerHeight = window.innerHeight / canvasScaleRatio;

// getBoundingClientRect() 메서드(화면상에 있는 오브젝트의 크기와 위치를 가져올 수 있다.) 이용
// 캔버스의 탑 위치
// 초기값이 0(false) => ! (true) : 처음에는 실행 값이 있을 경우 false
// 문제점: 스크롤 속도에 따라 값이 바뀐다. 
if (!values.rectStartY) { 
    values.rectStartY = objs.canvas.getBoundingClientRect().top;
    values.rect1X[2].end = values.rectStartY / scrollHeight;
    values.rect2X[2].end = values.rectStartY / scrollHeight;
}

const whiteRectWidth = recalculatedInnerWidth * 0.15;
values.rect1X[0] = (objs.canvas.width - recalculatedInnerWidth) / 2;
values.rect1X[1] = values.rect1X[0] - whiteRectWidth;
values.rect2X[0] = values.rect1X[0] + recalculatedInnerWidth - whiteRectWidth;
values.rect2X[1] = values.rect2X[0] + whiteRectWidth;

// 좌우 흰색 박스 그리기
objs.context.fillRect(values.rect1X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight); // 정지 값
objs.context.fillRect(values.rect2X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight);

break;

```
>> #### 정지상태 => 애니메이션
```
// objs.context.fillRect(values.rect1X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight); // 정지 값
// objs.context.fillRect(values.rect2X[0], 0, parseInt(whiteRectWidth), recalculatedInnerHeight);


// 애니메이션
objs.context.fillRect(
    parseInt(calcValues(values.rect1X, currentYOffset)),
    0,
    parseInt(whiteRectWidth),
    objs.canvas.height
);
objs.context.fillRect(
    parseInt(calcValues(values.rect2X, currentYOffset)),
    0,
    parseInt(whiteRectWidth),
    objs.canvas.height
);
```
>> #### 스크롤 바가 있을 경우 스크롤바가 영역을 차지 한다. innerWidth는 스크롤바를 포함한 영역이다. 
>> #### => recalculatedInnerWidth 값을 구할때, document.body.offsetHeight 사용한다.
```
// const recalculatedInnerWidth = window.innerHeight / canvasScaleRatio;
const recalculatedInnerWidth = document.body.offsetWidth / canvasScaleRatio;
const recalculatedInnerHeight = window.innerHeight / canvasScaleRatio;
```
>> #### offsetTop() 문서의 제일 상단을 기준으로 한다.
>> #### 하지만, 기준점을 바꿀 수 있다. => CSS로 속성값을 바꾼다.(static이 아닌 값)
```
CSS
.scroll-section {
    position: relative;
    padding-top: 50vh;
}
```
>> #### transform으로 줄이거나, 늘렸기 때문에 값이 맞지 않다. offsetTop()은 transform이 되기전(원본) 크기대로 계산한다.
>> #### scale 조정 이후의 값을 구한다.
```
// values.rectStartY = objs.canvas.getBoundingClientRect().top;
values.rectStartY = objs.canvas.offsetTop + (objs.canvas.height - objs.canvas.height * canvasScaleRatio) / 2;
```
>> #### 중간 지점에서(시작점) 애니메이션 실행
```
values.rect1X[2].start = (window.innerHeight / 2) / scrollHeight
values.rect2X[2].start = (window.innerHeight / 2) / scrollHeight
```
>> #### 박스 색 변경하기
```
objs.canvas.style.transform = `scale(${canvasScaleRatio})`;
objs.context.fillStyle = 'white';
objs.context.drawImage(objs.images[0], 0, 0);
```
>> #### 캔버스가 갑자기 등장 => 2번씬이 끝나갈때 3번씬을 미리 그려준다.
```
if (scrollRatio > 0.7) {
    console.log('0.7 start')
    const objs = sceneInfo[3].objs; // sceneInfo[3] 변수를 사용하게 처리
    const values = sceneInfo[3].values; // sceneInfo[3] 변수를 사용하게 처리
    const widthRatio = window.innerWidth / objs.canvas.width;
    const heightRatio = window.innerHeight / objs.canvas.height;
    let canvasScaleRatio;
    
    if (widthRatio <= heightRatio) {
        canvasScaleRatio = heightRatio;
    } else {
        canvasScaleRatio = widthRatio;
    }

    objs.canvas.style.transform = `scale(${canvasScaleRatio})`;
    objs.context.fillStyle = 'white';
    objs.context.drawImage(objs.images[0], 0, 0);

    const recalculatedInnerWidth = document.body.offsetWidth / canvasScaleRatio;
    const recalculatedInnerHeight = window.innerHeight / canvasScaleRatio;

    // 애니메이션 시점 필요가 없다.
    
    const whiteRectWidth = recalculatedInnerWidth * 0.15;
    values.rect1X[0] = (objs.canvas.width - recalculatedInnerWidth) / 2;
    values.rect1X[1] = values.rect1X[0] - whiteRectWidth;
    values.rect2X[0] = values.rect1X[0] + recalculatedInnerWidth - whiteRectWidth;
    values.rect2X[1] = values.rect2X[0] + whiteRectWidth;

    // 고정값으로 초기값만 가지고 있으면 된다.
    objs.context.fillRect(
        parseInt(values.rect1X[0]),
        0,
        parseInt(whiteRectWidth),
        objs.canvas.height
    );
    objs.context.fillRect(
        parseInt(values.rect2X[0]),
        0,
        parseInt(whiteRectWidth),
        objs.canvas.height
    );
}
```

<br>

### 이미지 블랜딩 => 마지막 씬 스텝으로 설정 (상황에 따라 나누어 준다.)
```
CSS
.image-blend-canvas.sticky {
    position: fixed;
    top: 0;
    margin: 0;
}


// 캔버스가 브라우저 상단에 닿는지
if (scrollRatio < values.rect1X[2].end) { // 애니메이션 end시점 이전이라면 
    step = 1;
    objs.canvas.classList.remove('sticky');
} else {
    step = 2;
    objs.canvas.classList.add('sticky');
    objs.canvas.style.top = `${-(objs.canvas.height - objs.canvas.height * canvasScaleRatio) / 2}px`;
}
```
> ### imageBlendY 추가
```
values: {
    rect1X: [0, 0, { start: 0, end: 0 }],
    rect2X: [0, 0, { start: 0, end: 0 }],
    imageBlendY: [0, 0, { start: 0, end: 0 }],
    rectStartY: 0,
}
```
> ### setCanvasImages()에서 씬3의 이미지를 세팅하기 때문에 씬3으로 접근
>> ### 이미지 셋팅을 했으나 아랫쪽이 짤려서 그려진다(위에서부터 그리기 때문에)
>> ### 그릴 이미지를 아래서부터 h(높이)값을 늘려주면서 y(섹션의 위치)를 바꿔주면 된다.

```
imageBlendY => BlendHeight [ 초기값:0, 최종값: 캔버스 높이, { start: 이미지가 fixed가 되는 순간, end:  } ]
초기값: 처음에는 안그려지고 조금씩 그려지면 된다.
case 3:

if (scrollRatio < values.rect1X[2].end) { // 애니메이션 end시점 이전이라면 
    step = 1;
    // console.log('캔버스 닿기 전')
    objs.canvas.classList.remove('sticky');
} else {
    step = 2;

    values.blendHeight[0] = 0;
    values.blendHeight[1] = objs.canvas.height;
    values.blendHeight[2].start = values.rect1X[2].end;
    values.blendHeight[2].end = values.blendHeight[2].start + 0.2;
    const blendHeight = calcValues(values.blendHeight, currentYOffset);

    objs.context.drawImage(objs.images[1],
        0, objs.canvas.height - blendHeight, objs.canvas.width, blendHeight,
        0, objs.canvas.height - blendHeight, objs.canvas.width, blendHeight)

    objs.canvas.classList.add('sticky');
    objs.canvas.style.top = `${-(objs.canvas.height - objs.canvas.height * canvasScaleRatio) / 2}px`;
```
> ### 이미지가 줄어드는 애니메이션 => 스크롤 값에 스케일 줄인다.
> ### 브라우저의 폭에 반응(내맘대로), 분모에 1.5를 곱해 값을 작게 만들어준다.
> ### start 후에 씬에 해당하는 20%동안 재생 => 이미지블렌드 후 축소되는 구간까지 전체 스크롤 구간의 총 40%

```
// 추가
canvas_scale: [ 0, 0, { start: 0, end: 0 } ],

// 스케일
if (scrollRatio > values.blendHeight[2].end) {
    values.canvas_scale[0] = canvasScaleRatio;
    values.canvas_scale[1] = document.body.offsetWidth / (1.5 * objs.canvas.width);
    // console.log(values.canvas_scale[1])
    values.canvas_scale[2].start = values.blendHeight[2].end;
    values.canvas_scale[2].end = values.canvas_scale[2].start + 0.2;

    objs.canvas.style.transform = `scale(${calcValues(values.canvas_scale, currentYOffset)})`
}
```
> ### 스케일 줄어든 후
> ### fixed라서 계속해서 쫓아다닌다 없앨 경우 위로 올라가버린다.(원래 위치) => 위치를 다시 잡아준다.(margin-top) => 캔버스가 fixed가 된 후 스크롤 된 만큼 margin-top
>> ### values.canvas_scale[2].end의 값이 위의 if문을 돌기전에 시작하여 값은 0 =>  && values.canvas_scale[2].end > 0 
```
스케일 if 문에
objs.canvas.style.marginTop = 0; 추가


if (scrollRatio > values.canvas_scale[2].end && values.canvas_scale[2].end > 0) {
    objs.canvas.classList.remove('sticky');
    objs.canvas.style.marginTop = `${scrollHeight * 0.4}px`;
}
```
> ### 문장(canvasCaption) opacity, translateY, 타이밍
```
CSS
.canvas-caption { margin: -8em auto 0; } 추가

values에 값 추가
canvasCaption_opacity: [ 0, 1, { start: 0, end: 0 } ],
canvasCaption_translateY: [ 20, 0, { start: 0, end: 0 } ],

시작지점이 이전씬의 마지막이라 이전의 if문에서 시작
values.canvasCaption_opacity[2].start = values.canvas_scale[2].end;
values.canvasCaption_opacity[2].end = values.canvasCaption_opacity[2].start + 0.1;
values.canvasCaption_translateY[2].start = values.canvasCaption_opacity[2].start;
values.canvasCaption_translateY[2].end = values.canvasCaption_opacity[2].end;

objs.canvasCaption.style.opacity = calcValues(values.canvasCaption_opacity, currentYOffset);
objs.canvasCaption.style.transform = `translate3d(0, ${calcValues(values.canvasCaption_translateY, currentYOffset)}%, 0)`;
```
> ### 텍스트(mid-message) 폭 수정 / 본문 패딩 제거(메뉴와 맞추기 위해)
```
폭 수정
.mid-message {
    width: 1000px;
}

패딩 제거
.description { padding: 0; }
mid-message { padding: 0; }
.canvas-caption { padding: 0; }
```

<br>

### 메뉴
```
CSS
.local-nav-sticky .local-nav {
    position: fixed;
    top: 0;
    background: rgba(255,255,255, 0.1);
    backdrop-filter: saturate(180%) blur(15px);
}
```
> ### checkMenu() 함수 추가
```
// 헤더, 첫번째 섹션의 높이(44px)
function checkMenu() {
    if (yOffset > 44) {
        document.body.classList.add('local-nav-sticky');
    } else {
        document.body.classList.remove('local-nav-sticky');
    }
}
```
> ### scroll함수에서 실행해준다.
```
window.addEventListener('scroll', () => {
    yOffset = window.pageYOffset;
    scrollLoop(); 
    checkMenu();
});
```

<br>

### 부드러운 감속의 원리 (스크롤 시 애니메이션 엑션을 조금더 부드럽게 하기)
> ### 첫번째 씬, 세번째 씬 그림을 그리는 부분
>> ### 똑같이 기본적인 틀을 잡아준다.
```
변수 추가
let acc = 0.1;
let deleayedYOffset = 0;
let rafId;
let rafState;

loop함수 추가
pageYOffset를 이미 변수 yOffset로 사용중이라 통일성 있게 변경
function loop() {
    deleayedYOffset = deleayedYOffset + (yOffset - deleayedYOffset) * acc;
    box.style.width = `${deleayedYOffset}px`;

    rafId = requestAnimationFrame(loop);

    if (Math.abs(yOffset - deleayedYOffset) < 1) {
        cancelAnimationFrame(rafId);
        rafState = false;
    }
}

scroll함수에 if문 추가
window.addEventListener('scroll', () => {
    yOffset = window.pageYOffset;
    scrollLoop(); // 스크롤 시 바로 실행되도록
    checkMenu();

    if (!rafState) {
        rafId = requestAnimationFrame(loop);
        rafId = true;
    }
});
```
>> ### playAnimation()에서 직접 그리는 sequence, drawImage부분 => loop()로 이동
>> ### 또, 필요가 없는 씬에서 작동할 필요가 없어서 조건을 걸어준다.
>> ### currentYOffset, objs, values는 playAnimation함수기에 똑같이 가져와준다.
```
const currentYOffset = yOffset - prevScrollHeight;
const objs = sceneInfo[currentScene].objs;
const values = sceneInfo[currentScene].values;

let sequence = Math.round(calcValues(values.imageSequenece, currentYOffset)); // 이미지 순서
objs.context.drawImage(objs.videoImages[sequence], 0, 0); // 이미지, x, y
```
>> ### currentYOffset의 yOffset을 deleayedYOffset으로 변경
```
const currentYOffset = deleayedYOffset - prevScrollHeight;
```
>> ### // 아래애서 위로 스크롤 시 에러
```
if (currentScene === 0) {
    console.log('loop'); // 동작 확인
    let sequence = Math.round(calcValues(values.imageSequenece, currentYOffset));
    // 아래애서 위로 스크롤 시 에러
    if (objs.videoImages[sequence]) {
        objs.context.drawImage(objs.videoImages[sequence], 0, 0);
    }
}
```
>>> ### 씬이 바뀌는 순간 첫번째 그림이 그려지는것 같은 부자연스러움 => enterNewScene : 씬이 바뀔때 계산 착오로 에러로 잠깐 스킵
>>> ### true일때 실행이 아니라, false일때 실행
```
if (!enterNewScene) {
    if (currentScene === 0) {
        const currentYOffset = deleayedYOffset - prevScrollHeight;
        const objs = sceneInfo[currentScene].objs;
        const values = sceneInfo[currentScene].values;
        let sequence = Math.round(calcValues(values.imageSequenece, currentYOffset));
        // 아래애서 위로 스크롤 시 에러
        if (objs.videoImages[sequence]) {
            objs.context.drawImage(objs.videoImages[sequence], 0, 0);
        }
    }
}
```
>>> ### 씬2번
```
case 2:
// let sequence2 = Math.round(calcValues(values.imageSequenece, currentYOffset)); // 이미지 순서
// objs.context.drawImage(objs.videoImages[sequence2], 0, 0); // 이미지, x, y

if (currentScene === 0 || currentScene === 2) 로 변경
```
>>> ### 중간중간에 다른 이미지가 나온다. => yOffset을 deleayedYOffset 변경
```
if (deleayedYOffset > prevScrollHeight + sceneInfo[currentScene].scrollHeight) {
    enterNewScene = true;
    currentScene ++;
    document.body.setAttribute('id', `show-scene-${currentScene}`);
}

if (deleayedYOffset < prevScrollHeight) {
    if (currentScene === 0) return; // 브라우저 최상단부분 바운스 효과로 인해 마이너스가 되는 것을 방지(모바일)
    enterNewScene = true;
    currentScene --;
    document.body.setAttribute('id', `show-scene-${currentScene}`);
}
```
>>> ### 
