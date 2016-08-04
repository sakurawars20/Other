# 筆記整理

### Rails架構小記

##### config folder內容

###### enviroments folder
 - 不同環境的設定檔，預設環境有
   - development：開發模式，用於開發時所使用的環境
   - test：測試模式，用於執行測試程式時所使用的環境
   - production：正式上線模式，用於正式上線時所使用的環境
 - initializers folder
   - (非rails核心的設定)在rails啟動時會自動載入執行
 - serects.yml
   - secrect_key_base：用來encode須保護的cookie訊息
   - 也可以用來存放第三方用的key or token
   - 需注意不要將這個檔案公開(加到.gitignore中)
 - route.rd
   - 路由設定
   - root慣例放在最下方
 - puma.rb
   - puma是rails5內建的app server，而puma.rb是他的設定檔
 - application.rb
   - 啟動rails程序時會執行application.rb的應用程式設定
 - boot.rb
   - 設定Gemfile的位置
   - 若Gemfile存在，則需要require 'bundler/setup'來設定所有RubyGem的載入路徑
 
###### Rake
  - Ruby版本的Make，可用來編寫script，設定相依性，以及執行順序

###### Rails default gem
  - rails
    - Ruby on Rails, full-stack web framework
  - sqlite3
    - rails內建的database，一般用於開發以及測試，不推薦用在正式上線版本(可用MySQL or PostgreSQL)
  - puma
    - rails5內建的app server
  - sass-rails
    - rails內建的sass整合引擎
      - (sass: Syntactically Awesome Style Sheets, 用來寫css的程式語言)
  - uglifier
    - ruby版本的JavaScript compressor
  - coffee-rails
    - CoffeeScript -> JavaScript adapter
  - jquery-rails
    - jQuery for rails
  - turbolinks
    - 使用Ajax的原理來加速網頁載入速度(只將body替換掉，而不重新讀取head及其他重複的部分)
    - 詳情請參考[turbolinks]
  - jbulider
    - 用於產生json格式的rails引擎
    - 詳情請參考[jbuilder]
  - byebug
    - ruby debugger
  - web-console
    - IRB console on browser
    - 在exception pages產生時，或是在code中插入<%= console %>，可以叫出這個IRB console
    - 詳情請參考[web-console]
  - listen
    - 監聽檔案是否有被修改，並且在檔案被修改時送出notification
  - spring
    - rails application preloader, speed up development by keeping application running in the background
    - 詳情請參考[spring]
  - spring-watcher-listen
    - Make spring watch the filesystem for changes using "listen" rather than by polling the filesystem
  - tzinfo-data (only use on windows)
    - IANA Time Zone Database
    - 若有這個package，tzinfo會使用這個package做為source of time zone data；若無則使用system zoneinfo files

### DOM
  - Document object Model
  - 將html， xml， xhtml文件中的標籤視為物件，並提供API可讓程式追蹤及修改(大部分是使用JavaScript，但不限)

### Reverse proxy server
  - [wikipedia][reverse_proxy]
  - 收到client送出的request後，收集client的資料，並轉送給internal network server，client並不知道internal network的存在
  - 用途
    - 隱藏origin web server的存在
    - Application firewall，可保護origin server免於一些常見的網路攻擊，比如DOS，DDOS
    - SSL加速(hardware acceleration)
    - 分配network loading。每個origin server有其服務的application，reverse proxy server可藉由rewrite URL的方式讓request對應到相對的web server
    - 減少origin server loading by caching static/dynamic content，也稱為web acceleration
    - Optimize content by compressing it，加速loading time
    - 動態生成的網頁可以一次生成後給reverse proxy server，再回覆給client。如此一來origin web server上生成網頁的program不需要remain open，從而release server resource
    - 可用於多個web server需被single publlic IP address訪問時。這些web server可能listen on different port in the same machine with same local IP address，或是different machine and different local IP address。Reverse proxy server分析這些incoming request之後轉發給right server within the local area network
    - 可實施A/B testing and multivariate testing，不需在網頁中放JavaScript tags or code
    - Commercial or enterprise level out-of-box solution(ex: McAfee Web Protection Gateway - SaaS solution)
    - 可提供基本的HTTP access authentication

### Web Server vs Application Server
 - Web Server：接受http request並產生http response回覆，such as HTML page
 - App server：處理商業邏輯
 - 單Web Server的情況：接受http request後，查詢DB，產生HTML回覆
   - 缺點：無法重複使用商業邏輯部分
 - Web Server-App Server架構：web server接收http request之後，pass商業邏輯部分給app server，回傳後再組成HTML回覆
   - 可重複利用商業邏輯部份給不同的應用
   - 讓擅長處理http request的web server處理http request，app server則focus在處理網站邏輯部份，提升效能
   - 也可以做到app server high availability and load balance
 - 近年來web server也可以做到類似app server的功能(XML payload)，app server也可能有web server as a subset
 - [Reference][web_app_server]

### Puma vs Unicorn
 - Puma
   - Multi-threaded
   - Bind ports or pull from a socket
   - Open a new thread for each incoming request
   - Doesn't maintain a master process
   - Runs best under JRuby or Rubunius(because of its multithread architecture)
   - Suited to single applications running on a host
 - Unicorn
   - Multi-processed
   - Pull from a socket
   - Start with a main process, then fork itself into multiple processes as workers
   - Main process manages other child process, kill worker if it takes too long time to execute a request, then fork itself as a new worker
   - Zero downtime deploy(master reload itself and its workers base off most recent code deploy)
   - Best use case: a single, dedicate application running at all time behide a reverse proxy and load balancer. It processes request very fast and has highly stable architecture
   - [Reference][puma_unicorn]

### Why rails5 switch web server from Webrick to Puma?
  - Rails5 use Action Cable for handling WebSockets, and Action Cable runs in-process with the rest of app(Webrick is a single-process, single thread app server)

### How browser works?
 - Main flow:
   1. Parsing HTML to construct the DOM tree
   2. Render tree construction
   3. Layout the render tree
   4. Painting the render tree
  
 1. 若parsing時遇到script，parser會立即停止解析網頁，並開始執行script的內容。若script為外部資源，則會從網路同步獲取資源完才繼續。若script屬性為defer，則會等HTML parsing結束才執行。
 2. 當DOM tree在建立的時候，會同時建立render tree，是由可視化元素依順序建立的tree，目的是按照正確的順序繪製內容。Renderer對應DOM元件，但並非一一對應，Non-visual DOM元件不會插入render tree中(display value = none者也不會，但hidden者會出現在tree中)
 3. Renderer添加到render tree時並不包含位置與大小，計算這些值的階段稱為layout or reflow。HTML uses a flow layout model，意即大部分情況只需一次single pass便能計算出所有幾何特徵(由左而右，由上而下)
 4. In painting step, render tree is traversed and renderer's `paint()` method os called to display content on screen.

 - [Reference][how_browser_works]

### Client端JavaScript timeline
 1. Browser建立Document，parsing網頁，並且將Element以及Text node加入Document中。*document.readyState = loading*
 2. 沒有async或是defer屬性的script：將這些元素加入文件中，並開始以同步的方式執行。Parser將會停止。script可以使用`document.write()`來將text插入input stream中。這類script可以看見本身擁有的script元素，以及在這之前的document內容。
 3. 屬性為async的script：browser會下載這個script，並且繼續parse Document。當script下載完畢，會盡快執行這個script，但parser不會停下來等script下載完畢。這類script無法使用`document.write()`，可以看見本身擁有的script元素，以及在這之前的document所有元素。可能可以，也可能無法存取剩餘的文件內容。
 4. 當文件parse完成，*document.readyState = interactive*
 5. 依序執行屬性為defer的script。async script也有可能在此時被執行。這類script可以看見完整的document tree，但無法使用`document.write()`
 6. Browser在Document物件上觸發DOMContentLoaded事件，從同步的script執行狀態轉變成非同步的事件驅動狀態。注意此時有可能仍有async script尚未被執行。
 7. 文件被完全parse，但browser可能在等待須載入的內容，如圖片。當所有內容載入完畢，且所有async script也執行完畢，*document.readyState = complete*。Browser會在Window物件上觸發load事件
 8. 事件處理器會被非同步的調用以回應使用者輸入事件，網路事件，計時器期滿等事件。

 - *感謝東霖的書籍贊助*

### JavaScript的一些特性整理

##### closure
 - 擁有閒置變數(free variable)的運算式。free variable依當時語彙環境而定。支援closure的語言通常具有First-class function。
 - Free variable：對函式而言非local variable也非parameter的變數，是綁定變數(Bound variable)。例如：
``` javascript
function doSome() {
  var x = 10;
  function f(y) {
    return x+y;
  }
  return f
}

> var foo = doSome()
> foo(20);  // 30
> foo(30);  // 40
```
 - closure關閉的是變數，不是變數所參考的值，例如：
``` javascript
function doOther() {
  var x = 10;
  function f(y) {
    return x+y;
  }
  x = 100;  
  return f
}

> var foo = doOther()
> foo(20);  // 120
> foo(30);  // 130
```
 - 也可以在closure中改變變數的值
``` javascript
var sum = 0;
[1, 2, 3, 4, 5].forEach(function(element) {
  sum += element;
});

sum; // 15
```

##### callback
 - 將A函式傳給B函式做為callback function，讓B函式執行完時，可回呼給A函式，例如：
``` javascript
function do_a(callback) {
  // do A
  callback && callback();
}

function do_b() {
  // do B
}

do_a(function() {
  do_b();
});
```
 - 可確保B函式的執行順序會在A函式執行完畢之後

##### async
  - 一個任務分成兩段，先執行第一段，轉而執行其他任務，在第一段執行完做好準備時，再執行第二段
  - 例如讀取文件進行處理：
    1. 向OS發出請求
    2. process執行其他任務
    3. OS返回document
    4. 繼續進行文件處理
 - 在JavaScript中以callback實現
 - 多個callback會造成callback hell，可以用Promise, Generator(ES6)，或是async(ES7)解決

### rails5使用PostgreSQL的步驟
 1. 安裝PostgreSQL
    - ubuntu上輸入`sudo apt-get install postgresql postgresql-contrib libpq-dev`
 2. 建立PostgreSQL user
    - `sudo -u postgres createuser -s [username]`
    - 若需更改密碼，可進入PostgreSQL console
    - `sudo -u postgres psql`
    - `postgres=# \password [username]`
 3. rails db設定：
    - Gemfile加入`gem 'pg'`，`bundle install`
    - 如要建立使用PostgreSQL的新project，輸入：`rails new [project_name] -d postgresql`
    - 原有的project改用PostgreSQL：
      - 更改database.yml設定：
      - `adapter: postgresql`
      - `encoding: unicode`
      - `username: [username]`
      - `password: [password]`
      - `host: localhost`
      - 注意database filename不要有'.'
 4. `rake db:setup`
      

   [turbolinks]: <https://github.com/turbolinks/turbolinks>
   [jbuilder]: <https://github.com/rails/jbuilder>
   [web-console]: <https://github.com/rails/web-console>
   [spring]: <https://github.com/rails/spring>
   
   [reverse_proxy]: <https://en.wikipedia.org/wiki/Reverse_proxy>
   
   [web_app_server]: <http://www.javaworld.com/article/2077354/learn-java/app-server-web-server-what-s-the-difference.html>
   
   [puma_unicorn]: <https://blog.engineyard.com/2014/ruby-app-server-arena-pt1>
   
   [how_browser_works]: <http://www.html5rocks.com/en/tutorials/internals/howbrowserswork>
