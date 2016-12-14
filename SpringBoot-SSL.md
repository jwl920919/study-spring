[Back](https://github.com/songagi/study-spring/blob/master/README.md)

# 폴더 이동
```
cd %JAVA_HOME%/bin
```

1. Keystore 생성
```
keytool -genkey -alias nds -keyalg RSA -keystore nds.ks
```

2. Keystore로 부터 인증서 추출
```
keytool -export -alias nds -keystore nds.ks -rfc -file nds.cer
```
	
3. Trust store 생성
```
keytool -import -alias nds -file nds.cer -keystore nds.ts
```

4. Spring Boot SSL 설정
	
```
server.port=8090

# Keystore
server.ssl.key-store= /myfolder/nds.ks
server.ssl.key-store-password= <PASSWORD>
	
# Alias
server.ssl.key-alias= nds
server.ssl.key-password= <PASSWORD>

# Trust store 
server.ssl.trust-store= /myfolder/ca.ts
server.ssl.trust-store-password= <PASSWORD>
```
  
[Back](https://github.com/songagi/study-spring/blob/master/README.md)
