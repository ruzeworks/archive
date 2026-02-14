/get_info는 사용자가 입력한 userid를 그대로 사용해 서버에서 내부 API로 요청을 보냅니다.

info = requests.get(f'{API_HOST}/api/user/{userid}').text

내부 API는 127.0.0.1에서만 접근 가능하지만, 위 로직 때문에 서버가 대신 요청해 주므로 내부 엔드포인트를 우회하여 호출할 수 있습니다.

또한 프론트에서 submit 이벤트로 userid를 guest/admin >> 0/1로 강제 변환해 임의 입력을 막고 있으므로, 콘솔에서 해당 submit 이벤트를 제거한 뒤 userid에 ../flag를 넣어 요청합니다.

이때 내부 요청 경로가 /api/user/../flag 형태가 되면서 결과적으로 /api/flag가 호출되어 플래그가 출력됩니다(람쥐).
