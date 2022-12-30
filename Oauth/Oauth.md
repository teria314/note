# Oauth
1. Oauth의 역할
2. 등록
3. Resource Owner의 승인
4. Resource Server의 승인
5. access token 발급
6. refresh token
------------------

# 1. Oauth의 역할

- Client : 서비스를 resource server에 접속해서 정보를 가져감
- Resource Owner : resource의 소유자(user)
- Resource Server : 제어하고자하는 resource를 가지고 있는 서버
- Authorization Server : 인증과 관련된 처리를 전담

</br></br>

# 2. 등록

- client가 resource server를 이용하기 위해서 resource server의 승인을 받아야하는데 이를 등록이라 한다.
- 공통적으로 client id, client secret authorized, redirect urls을 받는다.
    * client id : 만들고 있는 application을 식별하는 식별자
    * client secret : 해당 client에 대한 비밀번호
    * Authorized redirect Urls : 권한을 부여하는 과정에서 authorized code값을 전달받을 주소

</br></br>

# 3. Resource Owner의 승인

- Resource Server가 제공하는 다양한 기능들 중 일부 기능만 사용하고자 할 때 이 기능 사용에 대해 resource owner에게 동의를 받아야 한다.
- resource owner은 resource server로 접속하게 되고 그에 따라 기능을 수행한 후 client id값을 대조한후 redirect_url또한 대조시킨다.
- 같지않다면 작업을 끝내고 모두 같다면 scope에 해당되는 권한을 client에게 부여할것인지 resource owner에게 허용여부를 묻는다.
- 허용했다면 resource server는 user id와 그 user id가 허용한 scope정보를 얻게된다.

# 4. Resource Server의 승인

resource server는 resource owner에 대한 허용여부와 그에 따른 user id, scope정보를 획득한 이후, client에게 승인하는 과정을 거치는데 그 과정은 다음과 같다.

</br>

- 이때 authorization code 라는 임시비밀번호를 생성하고 resource owner에게 전달한다.
- 이후 resource owner는 client로 접속하게 되는데 authorization code가 client로 전달된다.
- client는 resource server에 접속하고 client_id, client_secret, authorization_code, redirect url를 보낸다.
- Resource server는 전달받은 정보를 바탕으로 자신이 가진 데이터와 대조후 모두 일치하면 access token을 발급하는 단계로 진입한다.

</br></br>

# 5. access token 발급

- client와 resource server에 있는 authorization code를 지운다.
- resource server는 accesstoken을 발급하고 client에게 해당 accesstoken으로 응답한다.
- 이 후 client가 resource server로 accesstoken으로 접근하면 resource server는 자신이 가진 accesstoken을 찾고 동작하게 된다.

</br> </br>

# 6. refresh token

access token은 수명이 존재한다. 따라서 이 수명이 끝날때마다 access token을 다시 발급받는 과정을 거쳐야 되는 데 이는 상당히 번거로울 것이다. 이를 손쉽게 처리할 수 있도록 하는 것이 refresh token이다.

- client에게 access token을 발급할때 refresh token도 같이 발급하는 경우가 많다.
- 이후 client가 access token이 수명이 끝나 더이상 token으로 resource를 가져올 수 없게 되면 보관하고 있던 refresh token을 authorization server에게 전달하고 access token을 다시 발급받게 된다.

