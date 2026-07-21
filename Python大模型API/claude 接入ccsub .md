# claude code 接入ccsub详细教程
## 1.安装git
打开 PowerShell，输入：  

winget install Git.Git  

然后回车，等待安装完成，输入：  

git --version  

验证git安装成功（能输出版本号表示安装成功）  


## 2.安装node  
浏览器打开：  
https://nodejs.org/  

进入官网下载对应的node（我是windows系统，所以下载Windows Installer (.msi) 64 位 ）  
下载完成进行安装  
打开新的Powershell窗口，输入：  

node -v  
npm -v  

出现版本号表示安装成功  

## 3.安装claude code
终端输入：

npm install -g @anthropic-ai/claude-code

（我原本按照哔站教程输入的是irm https://claude.ai/install.psl|iex  
 但是报错  
<img width="1072" height="248" alt="image" src="https://github.com/user-attachments/assets/4c5174f4-084a-4fb9-98e9-d2045ddd36a3" />
）  
输入：  

--version  

正确返回版本号表示下载成功  

但是现在输入：  

claude  

终端报错  
<img width="1248" height="742" alt="image" src="https://github.com/user-attachments/assets/8cfa33c9-bac5-48bf-a663-73c31bbe7ec7" />  

 
## 4.下载ccswitch
安装包：https://github.com/farion1231/cc-switch/releases/download/v3.16.5/CC-Switch-v3.16.5-Windows.msi  
然后进行安装  
然后进行以下操作：
<img width="1388" height="282" alt="image" src="https://github.com/user-attachments/assets/a57e0a10-aa02-4f1c-a6c1-d52905cc73aa" />  
点击我知道了  
<img width="1374" height="834" alt="image" src="https://github.com/user-attachments/assets/c05b288a-da3e-4d99-9bd1-c40318d935bb" />  
然后选择供应商，我选择的是ccsub
<img width="1470" height="960" alt="image" src="https://github.com/user-attachments/assets/f3a38983-00a8-4f0b-94be-37b33bc21a79" />
需要填写api key，可以点击获取api key申请一个，然后填进去，然后点击下方蓝色“+添加”按钮  
<img width="1324" height="330" alt="image" src="https://github.com/user-attachments/assets/ccb9589c-10be-4cf0-acb6-2ea03f2c9d61" />
然后点击上方的这个图标  
<img width="398" height="120" alt="image" src="https://github.com/user-attachments/assets/0f68e873-ab92-4966-a14e-6ccf40a05bfc" />  
点击“设置”的图标  
<img width="516" height="108" alt="image" src="https://github.com/user-attachments/assets/04751d97-d6c0-4048-8778-3cf315fb493a" />  
点击“路由”，打开本地路由中的“在主页面显示本地路由开关”和“路由总开关”，打开路由启用中的“Claud”  
重新打开终端输入:  
claude  
<img width="1016" height="1200" alt="image" src="https://github.com/user-attachments/assets/d0029634-fbbd-4d0b-9f8b-323cb221c596" />  
成功解决问题











