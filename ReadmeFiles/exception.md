# Exception
Có 4 exception trong trang web bao gồm:
- AccountNameExistException: được trả về khi tên tài khoản đã tồn tại khi tạo tài khoản mới
- AuthenticationAccountException: được trả về khi đăng nhập sai tên tài khoản, mật khẩu, hoặc sai vai trò tài khoản (tài khoản user lại đăng nhập vào trang admin)
- PasswordDoNotMatchException: được trả về khi tạo tài khoản, Password và RepeatPassword không trùng khớp
- UserNotFoundException: kiểm tra xem tài khoản của user có tồn tại trên hệ thống hay không

Code:

Cấu trúc khai báo một Exception (ví dụ với PasswordDoNotMatchException):
```
package com.data.filtro.exception;

public class PasswordDoNotMatchException extends RuntimeException {
    public PasswordDoNotMatchException(String message) {
        super(message);
    }
}

```
Trong đó: PasswordDoNotMatchException là lớp con của RuntimeException; được sử dụng để biểu diễn các ngoại lệ (exceptions) liên quan đến lỗi trong quá trình thực thi (runtime) của chương trình.

Cách sử dụng:

AuthenticationAccountException

```
public Account authenticateUser(String accountName, String password) {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        Account tempAccount;
        try{
            tempAccount = getAccountByName(accountName.trim());
            if (tempAccount.getRoleNumber() != 3){
                throw new AuthenticationAccountException("Incorrect Account");
            }
            if (tempAccount != null) {
                if (passwordEncoder.matches(password, tempAccount.getPassword())) {
                    Account authenticateAccount = accountRepository.authenticate(accountName, tempAccount.getPassword());
                    return authenticateAccount;
                } else {
//                System.out.println("sai mat khau");
                    throw new AuthenticationAccountException("Incorrect Password!");
                }
            } else {
                throw new AuthenticationAccountException("Incorrect AccountName!");
            }
        } catch (Exception exception){
            throw new AuthenticationAccountException("Incorrect Account");
        }
    }
```
---
PasswordDoNotMatchException và AccountNameExistException

```
public void registerUser(String userName, String accountName, String email, String password, String repeatPassword) {

        if (accountService.checkAccountName(accountName)) {
            throw new AccountNameExistException("Tên tài khoản đã được đặt");
        }

        if (!password.equals(repeatPassword)) {
            throw new PasswordDoNotMatchException("Không đúng mật khẩu !");
        }

        User user = new User();
        user.setName(userName);
        user.setEmail(email);
        userRepository.save(user);
        Account account = new Account();
        account.setAccountName(accountName);
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String hashPassword = passwordEncoder.encode(password);
        account.setPassword(hashPassword);
        account.setCreatedDate(new Date());
        account.setRoleNumber(3);
        account.setStatus(1);
        user.setAccount(account);
        accountRepository.save(account);
        userRepository.save(user);
    }
```
---
UserNotFoundException

```
public void updateResetPasswordToken(String token, String email) {
        Account account;
        User user = userService.getUserByEmail(email);
        if (user != null) {
            account = getAccountByName(user.getAccount().getAccountName());
            account.setPasswordResetToken(token);
            accountRepository.save(account);
        } else {
            throw new UserNotFoundException("User not found with the email address: " + email);
        }
    }
```