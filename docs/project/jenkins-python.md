## Jenkins 에서 Python 연동

개발사와 퍼블리싱 사업부간에 협업 리소스 비용을 줄이기 위한 기능들 개발.

### 이미지 배너 교체

* 이미지 파일명 입력받기
* cdn과 연결된 ftp에 입력된 이미지 파일 탐색
* 제공된 개발사 API에 cdn url 파라미터로 요청
* API 응답에 따라 슬랙 API 호출

#### 젠킨스 파라미터 설정

![img-banner](https://user-images.githubusercontent.com/20632507/189065003-7a076188-9748-4a1a-aeb9-f15905a4c007.png)

#### 젠킨스 쉘스크립트로 실행될 파이썬 스크립트

```
#!/usr/bin/env python3
import sys
from ftplib import FTP
import os
import warnings
import urllib3
import json

filename = os.environ['FILENAME']
warnings.filterwarnings(action='ignore')

ftp_host = 'ftp-host'
ftp_user = 'user'
ftp_passwd = 'pwd'
ftp_dir = 'dir'

api_host = 'http://apiurl'
api_key = 'api-key'
api_type = 'ChangePopUpImg'
color = {'SUCCESS': '#1AF952', 'FAILED': '#F91A1A'}

slack_url = 'https://hooks.slack.com/services/key'

ftp = FTP(host=ftp_host)
ftp.login(user=ftp_user, passwd=ftp_passwd)

# 파일리스트 구하기
def get_ftp_file_list(dir):
   ftp.cwd(dir)
   ftp_file_list = []
   file_list = []

   try:
      ftp.dir(ftp_file_list.append)
      for ftp_file in ftp_file_list[2:]:
         file = ftp_file.split()
         file_list.append(file[8])
   except:
      print('FTP 디렉터리가 존재하지 않습니다.')
      sys.exit(1)

   return file_list

# API로 배너 파일명 업데이트 하기
def update_banner_name(filename, file_list):

    if filename in file_list:
        http = urllib3.PoolManager()
        cdn_url = f'https://cdn.co.kr/{ftp_dir}/{filename}'
        api_url = f'{api_host}?Key={api_key}&Type={api_type}&ImgUrl={cdn_url}'
        print(api_url)
        response = http.request('GET', api_url)
        retMsg = response.data.decode("utf-8")
        if response.status == 200:
            send_slack_webhook_msg(gen_slack_message(color['SUCCESS'], '[API Schedule] 이미지 배너교체', retMsg))
        else:
            send_slack_webhook_msg(gen_slack_message(color['FAILED'], '[API Schedule] 이미지 배너교체', retMsg))
            print(retMsg)
            sys.exit(1)
    else:
        send_slack_webhook_msg(gen_slack_message(color['FAILED'], '[API Schedule] 이미지 배너교체', 'FTP 서버에 파일이 없습니다'))
        print('FTP 서버에 파일이 없습니다.')
        sys.exit(1)

# 슬랙 API 설정
def gen_slack_message(type, title, text):
        body, body_channel, body_attachments = {}, {}, {}
        body['attachments'] = []
        body_attachments["color"] = type
        body_attachments["pretext"] = title
        body_attachments["text"] = text
        body['attachments'].append(body_channel)
        body['attachments'].append(body_attachments)
        return json.dumps(body).encode('utf-8')

# 슬랙 API Request
def send_slack_webhook_msg(req_body):    
    req_headers = {'Content-Type': 'application/json; charset=UTF-8'}
    http = urllib3.PoolManager()
    req = http.request('POST', slack_url, headers=req_headers, body=req_body)
    result = req.data.decode('utf-8')
    return result


file_list = get_ftp_file_list(ftp_dir)
update_banner_name(filename, file_list)
```

#### 슬랙메시지 알람 연동

![slack-msg](https://user-images.githubusercontent.com/20632507/190053902-dca73d9c-97c4-4edf-8224-e5057a6ad7e5.png)

### 게임서버 점검

* 점검메시지 입력받기
* 제공된 개발사 API에 점검메시지 및 약속된 파라미터 값으로 요청
* API 응답에 따라 슬랙 API 호출

#### 젠킨스 파라미터 설정

![maintenance](https://user-images.githubusercontent.com/20632507/189065057-e89d4293-07ce-468d-accf-43e94771e2fe.png)

#### 젠킨스 쉘스크립트로 실행될 파이썬 스크립트

```
#!/usr/bin/env python3
import os
import warnings
import urllib3
import json

warnings.filterwarnings(action='ignore')

notice = os.environ['notice']

api_host = 'http://apiurl'
api_key = 'api-key'
api_type = 'Maintenance'
status = {'OPEN': 0, 'CLOSE': 1}

color = {'SUCCESS': '#1AF952', 'FAILED': '#F91A1A'}
slack_url = 'https://hooks.slack.com/services/key'

# 서버점검 API 호출
def update_Maintenance(notice_text):
    http = urllib3.PoolManager()
    api_url = f'{api_host}?Key={api_key}&Type={api_type}&Status={status["CLOSE"]}&Message={notice_text}'
    print(api_url)
    response = http.request('GET', api_url)
    retMsg = response.data.decode("utf-8")
    if response.status == 200:
        send_slack_webhook_msg(gen_slack_message(color['SUCCESS'], '[API Schedule] 서버 점검', retMsg))
    else:
        send_slack_webhook_msg(gen_slack_message(color['FAILED'], '[API Schedule] 서버 점검', retMsg))

# 슬랙 API 설정
def gen_slack_message(type, title, text):
        body, body_channel, body_attachments = {}, {}, {}
        body['attachments'] = []
        body_attachments["color"] = type
        body_attachments["pretext"] = title
        body_attachments["text"] = text
        body['attachments'].append(body_channel)
        body['attachments'].append(body_attachments)
        return json.dumps(body).encode('utf-8')

# 슬랙 API Request
def send_slack_webhook_msg(req_body):    
    req_headers = {'Content-Type': 'application/json; charset=UTF-8'}
    http = urllib3.PoolManager()
    req = http.request('POST', slack_url, headers=req_headers, body=req_body)
    result = req.data.decode('utf-8')
    return result
    
update_Maintenance(notice)
```

#### 슬랙메시지 알람 연동

![slack-msg](https://user-images.githubusercontent.com/20632507/190053924-8d3d5dea-17fa-4204-b399-1417c40f73c8.png)