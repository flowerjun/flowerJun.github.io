[![Trumpia](https://trumpia.com/responsive/images/common/trumpia-logo-default.png "Trumpia")](https://trumpia.com/ 'Trumpia')

# Tech Group - Electron

## Electron이란
- 일렉트론은 Cheng Zhao가 개발한 오픈 소스 프레임워크
- 지금은 깃허브에 의해 개발
- 원래 웹 애플리케이션을 위해 개발된 프런트엔드와 백엔드 구성 요소를 사용하여 데스크톱 그래픽 사용자 인터페이스 애플리케이션의 개발 가능
- 백엔드로는 Node.js 런타임을, 프론트엔드로는 크로미엄을 사용
- 상용화된 프로그램

    ![atom](https://electronjs.org/node_modules/electron-apps/apps/atom/atom-icon-64.png "atom") Atom
    ![slack](https://electronjs.org/node_modules/electron-apps/apps/slack/slack-icon-64.png "slack") Slack
    ![vsc](https://electronjs.org/node_modules/electron-apps/apps/visual-studio-code/visual-studio-code-icon-64.png "vsc") Visual Studio Code
    ![postman](https://electronjs.org/node_modules/electron-apps/apps/postman/postman-icon-64.png "postman") Postman 
    ![gitHub](https://electronjs.org/node_modules/electron-apps/apps/github-desktop/github-desktop-icon-64.png "gitHub") GitHub Desktop
    ![twitch](https://electronjs.org/node_modules/electron-apps/apps/twitch/twitch-icon-64.png "twitch") Twitch
    ![skype](https://electronjs.org/node_modules/electron-apps/apps/skype/skype-icon-64.png "skype") Skype
    ![whatsApp](https://electronjs.org/node_modules/electron-apps/apps/whatsapp/whatsapp-icon-64.png "whatsApp") WhatsApp

등...
## 1. Project폴더 생성 후 package.json 작성하기

앞서 Node.js를 설치 하였기 때문에 **npm init** 명령어가 가능.
커맨드 창에서 프로젝트 경로로 진입 한 후 **npm init -y** 명령어로 기본 package.json 파일을 생성. 
-y 옵션은 기본값으로 package.json을 생성하는 옵션.

package.json을 생성한 후 아래와 같이 수정(주석 제외, 주석 포함시 에러발생).

**package.json**
```
     {
        "name": "T-Address book",
        "version": "1.0.0",
        "description": "Trumpia Address book",
        "main": "main.js",          //entry point
        "scripts": {
            "start": "electron ."   // run script
        },
        "author": "junho <junho.lim@mytrum.com>", // Required for Linux build
        "license": "ISC"
    }
```
* entry point : 제어가 운영 체제에서 컴퓨터 프로그램으로 이동하는 것을 말하며, 프로세서는 프로그램이나 코드에 진입해서 실행을 시작
* ISC : ISC License는 Internet Systems Consortium(ISC)에 허용된 free Software license

## 2. Electron 설치
Electron을 아래와 같이 설치합니다.

**Electron 설치**
```
     npm install --save-dev electron
```

## 3. main.js 작성
main.js 파일을 package.json파일과 같은 경로에 아래와 같이 작성합니다.

```javascript
// app모듈과, BrowserWindow 모듈 할당
const {app, BrowserWindow, ipcMain} = require('electron');
let win;

app.on('ready', () => {
    win = new BrowserWindow({
            width : 800
            , minWidth:330
            , height :600
            , minHeight: 450 
            , show: false
            , icon: __dirname + '/app/resources/installer/Trum.ico'
            , webPreferences :{ 
                defaultFontSize : 14
            }
    });
    
    win.once('ready-to-show', () => {
        win.show();
    });

    /*
    let child = new BrowserWindow({parent: win, modal: true, show: false});
    child.loadURL("https://trumpia.com/");
    child.once("ready-to-show", () => {
        child.show();
    });
    */

    // win.loadURL('file://' + __dirname + '/index.html');
    win.loadURL(`file://${__dirname}/app/index.html`); //작은 따옴표가 아닌  back stick 기호(tab키 위)

    win.on('closed', () => {
        win = null;
    });

    // 개발자 도구 오픈
    win.webContents.openDevTools();
});
```

## 4. index.html 작성
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Trumpia Address-book</title>
        <link rel="stylesheet" href="./css/jquery.flipster.min.css">
        <link rel="stylesheet" href="./css/styles.css">
        <link rel="stylesheet" href="./css/flipsternavtabs.css">
    </head>
    <body>
        <!--<h1>Trumpia Electron Sample!</h1>-->
        <h1><img src="http://trumpia.co.kr/contact/img/Trumpia_Logo_2015.png"></h1>
        <h2>Trumpia Address-book</h2>

        <div class="flipster">
            <ul></ul>
        </div>

        <p class="moveTrumpia"></p>

        <script>window.$ = window.jQuery = require('./js/jquery.min.js');</script> // Electron에서 jQuery 사용 할 수 있게 선언.
        <script src="./js/index.js"></script>
        <script src="./js/jquery.flipster.min.js"></script>
    </body>
</html>

```

## 5. index.js 작성
```javascript
const {ipcRenderer, shell} = require('electron');

ipcRenderer.send('trumpiaContacts', 'start');

let contacts;
ipcRenderer.on('loadContacts', (event, arg) => {
    if (arg) 
        makeAddr(arg);
        // 처음 실행 시 맨 앞으로 이동.
        $(".flip-nav-category:eq(0)").find("li.flip-nav-item:eq(0)").find("a").click();
});

$(function(){
    let $trumpia = $('.trumpiaLink');
    $trumpia.append('<a href="https://www.trumpia.com"><span>-Trumpia, </span></a>');
    $trumpia.append('<a href="http://www.trumpia.co.kr/contact/"><span>-Trumpia Contacts</span></a>');
    $trumpia.on("click", "a", (e) =>{
        e.preventDefault();
        let moveUrl = $(e.target).parent().attr("href");
        shell.openExternal(moveUrl);
    });
});

function makeAddr(data) {
    let ul = $(".flipster ul");

    for ( let i in data ) {
        let item = data[i];

        let li = $('<li><img /><a target="_blank"></a></li>');
        li.attr("data-flip-category", item.team);
        li.attr("title", item.name);
        li.attr("id", item.id);
        
        li.find('a')
            .attr('href', "")
            .html(item.name + "<br/>" + item.phone + "<br/>" + item.email + "<br/>" + item.skype);

        li.find('img')
            .attr('src', item.imgPath)
            .css({
                "width" : 200
            });

        ul.append(li);
    }

    $('.flipster').flipster({
        enableNav: true,
        style: 'carousel'
    });
}    
```

## 5. 실행
커맨드 창에서 아래의 명령어를 실행합니다.
    ```
    npm start
    ```

## 6. 배포 (Window 기준)
1. 어플리케이션 패키징(electron-pacakger)
 - Electron 프로그램을 .app 또는 .exe 와 같은 실행 파일로 만들어 줌
    ```
    $ npm install electron-packager --save-dev # npm 스크립트로 사용
    $ npm install electron-packager -g # 전역 모듈로 설치
    
    electron-packager ./ myApp --platform=win32 --arch x64 \ --out dist \ --prune
    ```

2. 어플리케이션 패키징(asar)
 - electron으로 제작한 프로그램을 사용하기 위해서는, node_modules 을 포함한 모든 파일을 옮겨주어야 한다. 간혹 Windows에서 node_modules 등에서 긴 경로 이름으로 인해 복사가 안되는 오류가 발생하는데 이와 같은 문제를 완화하기 위해 electron 어플리케이션 패키징을 할 때는 asar 아카이브로 패키징
 - asar를 사용하는 또 다른 이유는 소스코드 전체를 복사해서 배포하는 것과 별개로 asar 아카이브를 통해 어플리케이션 소스 코드가 사용자에게 노출되는 것을 방지 할 수 있기 때문이다. asar가 tar과 비슷한 포맷이기 때문에 내부 소스가 암호화되어 패키징 되는 것은 아니어서 코드를 보려면 어렵지않게 확인 할 수 있지만, 소스코드 전체를 폴더구조로 공유하는 것 
    ```
    $ npm install -g asar
    $ asar pack app app.asar
    ```

3. 설치파일 만들기: Windows
 - Windows Setup 파일을 만들어 줌.
    ```    
    $ npm install --save-dev electron-winstaller
    $ node installer.js
    ```


## 배포
 - script-flow
    ```
    # delete older files
    rmdir ./dist/trumpiaApp-win32-x64 (rm -rf dist)
    
    # windows exe pacakaging
    electron-packager ./ trumpiaApp --platform=win32 --arch x64 --out dist --prune
    
    # asar packaging
    asar pack ./dist/trumpiaApp-win32-x64/resources/app ./dist/trumpiaApp-win32-x64/resources/app.asar
    
    # delete source dir
    rmdir ./dist/trumpiaApp-win32-x64/resources/app
    (rm -rf ./dist/trumpiaApp-win32-x64/resources/app)
    
    # create installer
    node installer.js
    ```
### 참고 사이트 
#### [배포 참고 https://proinlab.com/archives/1928](https://proinlab.com/archives/1928)
#### [배포 참고 https://github.com/cionman/03_Electron_Distribution](https://github.com/cionman/03_Electron_Distribution) 
