"# crawling-with-nodejs"
### NodeJs 크롤링 - 파이썬, 자바 001
1. CSV, XLSX 파싱
1. axios, cheerio로 간단한 페이지 크롤링
1. puppeteer로 복작합 페이지 크롤링
  - SPA 크롤링 (인피니트 스크롤링)
  - 로그인이 필요한 페이지 스크롤링
  - 데이터 베이스에 저장
  - 크롤링한 결과물을 다시 CSV, XLSX로 저장
## 장점
1. 웹을 구성하는 언어가 자바스크립트기 때문에 호완성이 좋다.
  - 생산성이 좋다.

### csv-parse 패키지로 csv 파싱하기
3. npm init
3. csv: 콤마와 줄 바꿈의로 구별된 값들
3. npm i csv-parse 바퀴를 재발명하지 말아라
3. const parse = require('csv-parse/lib/sync')
  - node_modules/csv-parse/lib/sync 함수를 require 하는 것으로 보면됩니다.

### xlsx패키지로  엑셀 파싱하기
1. npm i xlsx

### axios + cheerio
1. npm i axios cheerio

### puppeteer 002
1. npm i puppeteer
2. userAgent 내 환경이 무엇인지
3. const browser = await puppeteer.launch({headless:false});
  - headless : 서버에서 사용하기 때문에 화면에 보이고 안보이고를 판단 할 수 있다
4. npm i csv-stringify
5. document 쓰는 위치 잘 기억!! 002/index.js
6. https://try-puppeteer.appspot.com
7. page.setUserAgent : navigator.userAgent

### axios & cheerio로 이미지 다루기 003
##### 이미지 선택자 CSS 선택자
1. $(querySelector) $$(querySelectorAll)
2. #(아이디) .(클래스)
  - $(div.poster)
  - $(div#container)
3. $$(div img)
  - 띄어쓰기([자손])
4. $$('div > a > img')
  - > (무조건 부모 []자식] 관계다) 깊이가 바로 아래
5. $$('div.poster>a>img')
6. class="poster post_left" => poster.post_left
7. $('img[width=30]') 속성으로 찾기

##### 이미지 다루기
<pre>
<code>
  const imgResult = await axios.get(data.img.replace(/\?.*$/, ''), {
    responseType:'arraybuffer',
  });
  fs.writeFileSync(`003/poster/${r[0]}.jpg`, imgResult.data);


  const buffer = await page.screenshot({
    path:`003/screenshot/${r[0]}.png`,
    //fullPage:true,
    clip:{
      /*
        0,0 [ 시작:(100,100 ), (width : 300,height: 300) ]
      */
      x:100,
      y:100,
      width:300,
      height:300
    }
  });
</code>
</pre>

### 인피니트 스크롤링 크롤링 004
<pre>
<code>
const puppeteer = require('puppeteer');
const axios = require('axios');
const fs = require('fs');

const getImages = async (page) => {
  await page.waitFor(2000);
  return page.evaluate(async ()=>{
    window.scrollTo(0,0);
    let imgs = [];
    const imgsEls = document.querySelectorAll('img._2zEKz');
    if(imgsEls.length){
      imgsEls.forEach(v => {
        let src = v.src;
        if(src) imgs.push(v.src);
        nowTag = v.parentElement.className
        v.parentElement.removeChild(v);
      })
    }
    window.scrollBy(0,1500);
    return imgs
  });
}

const crawler = async () => {
  try {
    let result = [];
    const browser = await puppeteer.launch({ headless:process.env.NODE_ENV === 'production'});
    const page = await browser.newPage();
    await page.goto('https://unsplash.com');
    while(true){
      result = result.concat( await getImages(page) );
      if(result.length > 10) break;
    }
    await page.close();
    await browser.close();
    return result;
  } catch (e) {
    console.error(e);
  }
}

(async () => {
  try {
    fs.readdir('004/images', err => err && fs.mkdirSync('004/images') );
    const result = await crawler();
    fs.writeFile('004/result.txt',result.join("\n"), error => {
      if(error)throw error;
    })
    result.forEach(async (src, index) => {
      const image = await axios.get(src, {responseType:'arraybuffer'});
      fs.writeFileSync(`004/images/${index}.jpeg`, image.data);
    });
  } catch (e) {
    console.error(e);
  }
})();

</code>
</pre>

### 페이스북 로그인 태그 분석 005
### 프록시 006
1. 프록시는 다른 사람의 아이피를 이용해서 요청을 보낼 수 있도록 해주는 (대리인)
2. 익명성이 항상 보장되지는 않습니다.
  - HIA, NOA,..
3. db 연결해서 찾은 데이터 저장

### 페이스북 크롤링 해보기 007
