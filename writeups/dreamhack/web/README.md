문제 파일을 보면

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'admin': FLAG
}


admin으로 로그인한다면 flag 값이 출력됩니다.
10번째 줄을 보면 guest는 guest로 로그인이 가능하므로, guest로 로그인합니다.

username = request.cookies.get('username', None)
if username == "admin":
    "flag is " + FLAG

쿠키의 username 값만 읽어서 admin인지 판단하므로, username의 value를 admin으로 바꾼 뒤 새로고침하면 됩니다.
