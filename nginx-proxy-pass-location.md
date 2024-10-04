NGINX로 proxy_pass 를 할 때 URI 경로에 /sample 과 /sample/ 사용에 따른 차이

NGINX에서 location과 proxy_pass 설정은 매우 중요하며, 특히 URI 경로(/api와 /api/)와 백엔드 서버로의 프록시 경로 (ddns와 ddns/)가 결합된 경우 그 동작 방식이 달라집니다. 아래에서 2x2의 조합을 설명하겠습니다.

1. location /api + proxy_pass ddns

이 경우는 /api로 시작하는 요청이 ddns로 프록시됩니다. 중요한 점은 location /api는 /api 자체와 /api/foo와 같은 요청들을 모두 매칭하지만, proxy_pass에서 상대경로인 ddns가 사용되면 location에서 매칭된 /api 부분이 proxy_pass에 추가되지 않고, 요청 URI가 그대로 전달됩니다.

    •	요청: /api/test
    •	프록시로 전달되는 URI: http://ddns/api/test

2. location /api/ + proxy_pass ddns

이 경우는 /api/로 시작하는 요청에만 매칭됩니다 (예: /api/foo, /api/), 하지만 /api로 끝나는 요청은 매칭되지 않습니다. 프록시 전달 방식은 ddns로 전달될 때 /api/ 경로는 그대로 유지되고 proxy_pass에서 추가되지 않습니다.

    •	요청: /api/test
    •	프록시로 전달되는 URI: http://ddns/api/test

3. location /api + proxy_pass ddns/

이 경우 /api로 시작하는 요청이 /api와 /api/foo를 포함해 매칭되며, 중요한 점은 proxy_pass에 /가 추가된 경우입니다. NGINX는 이 경우 URI를 처리하면서 location에서 매칭된 /api 경로 부분을 제거하고 나머지 URI를 ddns/ 뒤에 붙여서 전달합니다.

    •	요청: /api/test
    •	프록시로 전달되는 URI: http://ddns/test (여기서 /api 부분이 사라짐)

4. location /api/ + proxy_pass ddns/

이 경우는 /api/로 시작하는 요청만 매칭됩니다. URI에서 /api/ 부분은 proxy_pass에 포함되지 않고, ddns/로 전달됩니다. 이 동작 방식은 /api/를 처리하지 않고 나머지 경로를 백엔드로 넘겨줍니다.

    •	요청: /api/test
    •	프록시로 전달되는 URI: http://ddns/test

요약:

    •	location /api: /api와 /api/foo를 포함하는 요청들을 매칭합니다.
    •	location /api/: /api/로 시작하는 요청만 매칭되며 /api는 매칭되지 않습니다.
    •	proxy_pass ddns: 매칭된 URI 전체가 프록시 서버로 전달됩니다.
    •	proxy_pass ddns/: location에서 매칭된 경로는 제거되고 나머지 경로만 프록시 서버로 전달됩니다.

권장사항:

    •	REST API 경로 처리: 보통 /api/ 형태로 사용하고 proxy_pass에는 ddns/를 사용하는 경우가 많습니다. 이렇게 하면 /api/ 뒤의 경로만 백엔드 서버로 전달되므로 깔끔한 URI 처리가 가능합니다.

이 구조에서는 경로 설정에 따라 프록시 서버로 전달되는 URI가 달라지므로, 목적에 맞는 설정을 선택하는 것이 중요합니다.
