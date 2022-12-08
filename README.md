# naukma-aws

bla bla bla

# User service 
gRPC cервіс користувачів, виконує роль зберження користувачі та їх аутнефікації. 
## Сутності 
```protobuf
message User {
    string email = 1;
    string nickname = 2;
    string password = 3;
    string phoneNumber = 4;
    bool emailConfirmed = 5;
    string description = 5;
    bool isHidden = 6;
}
```
## routes
### POST: /api/user/auth/local/signup
Раут для створення користувача, приймає 
```protobuf
message UserSignupRequest {
    string email = 1;
    string nickname = 2;
    string password = 3;
    string phoneNumber = 4;
}
service UserService {
    rpc SigninRequest(SigninRequest) returns(JwtTokenPair);
}
```
Після збереження користувача в базу буде викликаний івент user.created
### PATCH: /api/user/{:id}
Раут для зміни профілю користувача 
```protobuf
message UserUpdateRequest {
    uuid userId = 1;
    string nickname = 2;
    string password = 3;
    string description = 5;
}
service UserService {
    rpc SigninRequest(UserUpdateRequest) returns(User);
}
```
Після збереження користувача в базу буде викликаний івент user.updated
### POST: /api/user/auth/local/signin
Стратегія аутнефікації така, використовується jwt токени, refresh та access, 
```protobuf
message JwtTokenPair {
    string accessToken = 1;
    string refreshToken = 2;
}
```
В access payload зберігаються такі поля як id користувача, його нікнейм, та sessionId
Цей sessionId буде поміщений в кеш сесії. Цей токен живе всього 15 хв і після 15 хв ключ з кешу буде інвалідовано.
Цей кеш спійльний разом з Edge service.
В refresh token генерується uuid та йде в базу даних разом з часом створення.
```protobuf
message SigninRequest {
   string email = 1;
   string password = 2; 
}
service UserService {
    rpc SigninRequest(SigninRequest) returns(JwtTokenPair);
}
```
### POST: /api/user/auth/refresh
Разом з рефреш токеном можна за допомогою refreshToken, така ж логіка наче ми наново авторизуємося, тільки на цей раз ми використовуємо refreshToken
```protobuf
message RefreshTokenRequest {
   string refreshToken = 1;
}
service UserService {
    rpc RefreshTokens(RefreshTokenRequest) returns(JwtTokenPair);
}
```
### DELETE: /api/user/{:id}
Раут з видаленням користувача, насправді не видаляє, а приховує його, також викликає SAGA транзацкію для приховання постів і коментів користувача

```protobuf
message DeleteUserRequest {
   string userId = 1;
}
service UserService {
    rpc DeleteUser(DeleteUserRequest) returns(bool);
}
```
- Edge service
Сервіс API Gateway, функції
- Авторизація користувача
Перевірка 


