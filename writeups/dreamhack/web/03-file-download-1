/read에서 name 파라미터로 받은 값을 파일 경로에 그대로 사용하니,

@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True
        
../가 필터링되지 않아 상위 디렉토리로 탈출이 가능함.

/read 엔드포인트로 접근한 후 upload 페이지를 read?name=../flag.py으로 수정 후 새로고침
