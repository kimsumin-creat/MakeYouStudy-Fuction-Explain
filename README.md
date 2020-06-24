
# MakeYouStudy-Fuction-Explain
# 3. 기능 구현
##   각 액티비티 기능 설명
|  클래스              | 기능                     |layout                         |
|----------------|-------------------------------|-----------------------------|
|Login           |  로그인                        |login.xml                       |
|                | 비밀번호 찾기                   | activity_find.xml              |
|                |회원가입                         |activity_signup.xml            |
|Logout          |책상이미지등록, 로그아웃, 회원탈퇴  |acticity_profile.xml           |
|Timetable       |시간표 생성                    |activity_timetable.xml|
|                |시간표 추가                      |activity_edit.xml              |
| Diary          |다이어리 작성                    |tab_write                     |
|                |다이어리 목록 보기               |list_layout.xml              |
|                |업데이트된 다이어리 글 보기         |activity_diary_update.xml   |
|Calendar        |캘린더 생성 및 작성               |acticity_calendar.xml |
|Attendance      |Image Matching 출석체크             |  activity_image_matching.xml      | 
|                |사물인식 출석체크                |activity_image_label.xml|
|                |Text인식 출석체크              |  activity_machine_learning.xml      |
|                | 출석률 그래프로 보기          |activity_attendance_rate.xml           |                

<br>

##  권한 허용 알림 
### Permission Check

Make You Study는 다음의 권한을 허용해 주어야 한다.

~~~java
// Image Machine Learning에 필요한 권한
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> // 파일 쓰기 권한
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" /> // 파일 읽기 권한
<uses-permission android:name="android.permission.CAMERA" /> // 카메라 권한
<uses-permission android:name="android.permission.RECORD_AUDIO"/> // 오디오 권한
// Alarm에 필요한 권한
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/> // 오버레이 권한
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" /> // Foreground service 권한
<uses-permission android:name="android.permission.WAKE_LOCK"/> // 절전모드 권한
<uses-permission android:name="android.permission.VIBRATE"/> // 진동 권한
// Firebase, Facebook, Google Login에 필요한 권한
<uses-permission android:name="android.permission.INTERNET"/> // 인터넷 접속 권한
~~~
위의 권한 중에서 사용자가 직접 허용해줘야 하는 권한이 있다.
> 권한을 [직접 허용해줘야 하는 이유](https://developer.android.com/training/permissions/requesting?hl=ko#perm-check)

아래 권한은 직접 사용자가 허용해줘야하는 권한들이다.
~~~java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> // 파일 쓰기 권한
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" /> // 파일 읽기 권한
<uses-permission android:name="android.permission.CAMERA" /> // 카메라 권한
<uses-permission android:name="android.permission.RECORD_AUDIO"/> // 오디오 권한
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/> // 오버레이 권한
~~~
#### MainActivity.Java 에서 권한 요청을 수행한다.
권한 요청이 필요한 권한들을 list에 담아준다.
~~~java
private static final int MULTIPLE_PERMISSIONS = 101; // 여러 권한을 요청하기 위한 code
    private String[] permission = {
            Manifest.permission.WRITE_EXTERNAL_STORAGE,
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO
    };
~~~
아래와 같이 권한을 확인하고 허용되지않은 권한은 요청할 수 있다.
~~~java
    private boolean checkPermission(){
        int result;
        List<String> permissionList = new ArrayList<>();
        for (String pm : permission){
            result = ContextCompat.checkSelfPermission(this, pm);
            if(result != PackageManager.PERMISSION_GRANTED){
                permissionList.add(pm);
            }
        }if(!permissionList.isEmpty()){
            ActivityCompat.requestPermissions(this, permissionList.toArray(new String[permissionList.size()]), MULTIPLE_PERMISSIONS);
            return false;
        }
        return true;
    }
~~~
위에서 **SYSTEM_ALERT_WINDOW** 권한은 따로 확인하지 않았는데 이는 사용자가 직접 설정창으로 가서 허용을 해줘야 한다. 이에 추가적으로 dialog로 사용자에게 권한을 허용해야하는 이유에 대해서 알리고, 권한을 요청한다.
~~~java
private void checkOverlayPermission(){
    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setTitle("다른 앱 위에 쓰기 권한").setMessage("Make You Study의 알람 화면을 띄우기 위해서 권한을 허용해 주셔야 합니다.");
    builder.setPositiveButton("확인", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            try{
                Uri uri = Uri.parse("package:" + getPackageName());
                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, uri);

                startActivityForResult(intent, 5469);
            }catch (Exception e){
                Log.d("MainActivity", "" + e);
            }
        }
    });
    builder.setNegativeButton("종료", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            showToast_PermissionDeny();
        }
    });
    AlertDialog alertDialog = builder.create();
    alertDialog.show();
}
~~~
아래와 같은 결과를 볼 수 있다.

![permissionCheck](https://user-images.githubusercontent.com/46085058/85485386-9473de00-b603-11ea-891a-c55566c952d5.png)

<br>


[AndroidManifest.xml전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/AndroidManifest.xml)


<br>

## 로딩 화면

어플이 실행 되면 가장 먼저 보여지는 화면이다.

#### Loading.java



Handler 를 이용해 3초의 딜레이 진행 뒤에야 MainActivity 화면으로 전환 된다.

 ```java
Handler timer=new Handler();
        timer.postDelayed(new Runnable()
        {
            @Override
            public void run() 
            {
                Intent intent=new Intent(getApplication(), MainActivity.class);
                startActivity(intent);
                finish();
            }
        },3000);
````

![loading](https://user-images.githubusercontent.com/50138845/85493659-431f1b00-b612-11ea-8688-b82e57a93f15.gif)


[Loading.java 전체 코드 보기 ](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/Loading.java)

## 메인 화면

#### MainActivity.java

로그인을 하면 나오는 화면이다.  로그인에 성공 하면 어플 실행에 필요한 권한을 요청한다.<br>
어플의 기능을 한곳에 모아  사용자가 필요로 하는 기능을 선택할 수 있다. MainActivity는 5개의 버튼으로 구성 되어 있다.
- Calnder 
- TimeTable
- Diary
- AttendanceRate
- Profile
<br>
순서대로 각 button의 icon image이다.

![fffff](https://user-images.githubusercontent.com/50138845/85494542-d60c8500-b613-11ea-80b1-c9d4917bbdd4.png)

<br>

### 버튼 클릭 Listener

Button이 클릭 되면 ,해당 Activity로 전환 된다.
   ```java
        {
    buttonCalendar.setOnClickListener(this);
    buttonTimeTable.setOnClickListener(this);
    buttonDiary.setOnClickListener(this);
    buttonProfile.setOnClickListener(this);
    buttonAttendanceRate.setOnClickListener(this);
    }
    @Override
    public void onClick(View view)
    {
        if(view == buttonCalendar){
        startActivity(new Intent(getApplicationContext(), CalendarActivity.class));
        }
        if(view == buttonTimeTable){
         startActivity(new Intent(getApplicationContext(), TimeTableActivity.class));
        }
        if(view == buttonDiary){
        startActivity(new Intent(getApplicationContext(), DiaryActivity.class));
        }
        if(view == buttonProfile){
        startActivity(new Intent(getApplicationContext(), ProfileActivity.class));
        }
        if(view == buttonAttendanceRate){
        startActivity(new Intent(getApplicationContext(), AttendanceRateActivity.class));
        }
    }
 
 
<br>


 어플 내 있는 기능 수행에 앞서 권한을 요청 한다.

 if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M){
            checkPermission();
            // Android 10 이상부터 사용자가 직접 OverlayPermission을 설정해 줘야함
            if(!Settings.canDrawOverlays(getApplicationContext())){
                checkOverlayPermission();
            }
        }
        //initializing views
        buttonCalendar = (Button)findViewById(R.id.buttonCalendar);
        buttonTimeTable = (Button)findViewById(R.id.buttonTimeTable);
        buttonDiary = (Button)findViewById(R.id.buttonDiary);
        buttonProfile = (Button)findViewById(R.id.buttonProfile);
        buttonAttendanceRate = (Button)findViewById(R.id.buttonAttendance);
        imageViewGood = (ImageView)findViewById(R.id.good);
        imageViewGood.setImageResource(res);
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode){
            case MULTIPLE_PERMISSIONS:{
                if(grantResults.length > 0){
                    for (int i = 0; i < permissions.length; i++){
                        if(permissions[i].equals(this.permission[i])){
                            if(grantResults[i] != PackageManager.PERMISSION_GRANTED){
                                showToast_PermissionDeny();
                            }
                        }
                    }
                }else{
                    showToast_PermissionDeny();
                }
                return;
            }
        }
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }
    private boolean checkPermission(){
        int result;
        List<String> permissionList = new ArrayList<>();
        for (String pm : permission){
            result = ContextCompat.checkSelfPermission(this, pm);
            if(result != PackageManager.PERMISSION_GRANTED){
                permissionList.add(pm);
            }
        }if(!permissionList.isEmpty()){
            ActivityCompat.requestPermissions(this, permissionList.toArray(new String[permissionList.size()]), MULTIPLE_PERMISSIONS);
            return false;
        }
        return true;
    }
    // alarm overlay permission check 알람이 시작될 때 Activity를 띄워줌
    private void checkOverlayPermission(){
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("다른 앱 위에 쓰기 권한").setMessage("Make You Study의 알람 화면을 띄우기 위해서 권한을 허용해 주셔야 합니다.");
        builder.setPositiveButton("확인", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                try{
                    Uri uri = Uri.parse("package:" + getPackageName());
                    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, uri);

                    startActivityForResult(intent, 5469);
                }catch (Exception e){
                    Log.d("MainActivity", "" + e);
                }
            }
        });
        builder.setNegativeButton("종료", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                showToast_PermissionDeny();
            }
        });
        AlertDialog alertDialog = builder.create();
        alertDialog.show();
    }
    // Permission check notification
    private void showToast_PermissionDeny() {
        Toast.makeText(this, "권한 요청에 동의 해주셔야 이용 가능합니다. 설정에서 권한 허용 하시기 바랍니다.", Toast.LENGTH_SHORT).show();
        finish();
    }
}
<br>

로그인이 되어 있어야 어플사용이 가능 하다.  로그인 되어 있지 않으면 로그인 화면으로 전환한다.<br>
  firebaseAuth.getCurrenter는  현재 로그인 되어있는 사용자를 의미 한다.
```java
 // 유저가 로그인하지 않은 상태라면 LoginActivity 실행
 firebaseAuth = FirebaseAuth.getInstance();
 if(firebaseAuth.getCurrentUser() == null)
        {
            finish();
            startActivity(new Intent(this, LoginActivity.class));
        }
        else{
            FirebaseUser user = firebaseAuth.getCurrentUser();
            //button event
            buttonCalendar.setOnClickListener(this);
            buttonTimeTable.setOnClickListener(this);
            buttonDiary.setOnClickListener(this);
            buttonProfile.setOnClickListener(this);
            buttonAttendanceRate.setOnClickListener(this);
        }
```
<br>

## Random wise saying

### MainActivity에 랜덤으로 명언띄우기

![명명언](https://user-images.githubusercontent.com/62635984/85487460-9049bf80-b607-11ea-857c-45234c4a0348.png)

#### MainActivity.java
MainActivity에 여러가지 명언 중 하나의 명언을 띄워줌으로써 공부에 대한 의지를 올린다.

배열에 명언을 적어놓은 image를 넣어주고, 랜덤값을 배열 값에 넣어 랜덤으로 하나의 배열을 res변수에 넣어준다.
~~~java
int index = (int) (Math.random() * 10);  
int res = ran[index];  
  
public static final int ran[]= {  
    R.drawable.good1, R.drawable.good2, R.drawable.good3,  
    R.drawable.good4, R.drawable.good5, R.drawable.good6,  
    R.drawable.good7, R.drawable.good8, R.drawable.good9, R.drawable.good10  
};

private ImageView imageViewGood;
~~~
ImageView에 res변수에 넣어준 배열에 해당하는 image를 띄워준다.
~~~java
imageViewGood = (ImageView)findViewById(R.id.good);  
imageViewGood.setImageResource(res);
~~~
<br>

[MainActivity.java 전체코드 보기 ](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/MainActivity.java)

<br>

## 로그인

![로로그인](https://user-images.githubusercontent.com/62635984/85486856-6e9c0880-b606-11ea-9c61-e55a34224c3f.png)

 맨 처음 어플을 실행 하였을때, 로그인이 되어 있지 않으면 로그인 화면으로 전환 된다.  <br>
 로그인 방법은 3가지가 있다. 3가지 방법은 화면 전환 없이 진행 된다.
 - 회원가입한 계정으로  로그인
 - 구글계정으로 로그인
 -  페이스북계정으로  로그인
 
 계정이 없거나, 계정이 있지만 비밀번호를 분실하였을 때 버튼을 클릭 하면 화면이 전환되고 이름에 맞게  기능을 수행 할 수 있다.
  - 회원가입
  - 비밀번호 찾기
   
<br>

### 로그인 방식

#### LoginActivity.java 

회원가입한 계정으로 로그인 
```java
 //로그인 버튼 클릭시 파이어베이스 eamil 로그인 시작
        btn_login.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                email = ed_eamil.getText().toString();
                password = ed_password.getText().toString();
                if (isValidEmail() && isValidPasswd()) {
                    loginUser(email, password);
                }
            }
        });
    }//이메일 유효성 검사
    private boolean isValidEmail() {
        if (email.isEmpty()) {
            // 이메일 공백
            Toast.makeText(LoginActivity.this,"이메일이 공백입니다.",Toast.LENGTH_SHORT);
            return false;
        }
        else {
            return true;
        }
    }
    // 비밀번호 유효성 검사
    private boolean isValidPasswd() {
        if (password.isEmpty()) {
            // 비밀번호 공백
            Toast.makeText(LoginActivity.this,"패스워드가 공백입니다.",Toast.LENGTH_SHORT);
            return false;
        } else {
            return true;
        }
    }
    //입력한 이메일 과 비밀번호에 오류가 없다면 createUser() 가 동작합니다.email과 password를 받아와  `createUserWithEmailAndPassword`에 전달하여 신규 계정을 생성합니다. 계정 생성에 성공을 하면 MainAcitivity로 화면이 전환 됩니다. 계정 생성 실패시  메세지를 띄웁니다.

    private void loginUser(String email, String password)
    {
        firebaseAuth.signInWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            // 로그인 성공
                            Toast.makeText(LoginActivity.this, R.string.success_login, Toast.LENGTH_SHORT).show();
                            Intent intent=new Intent(getApplicationContext(),MainActivity.class);
                            startActivity(intent);
                            finish();
                        } else {
                            // 로그인 실패
                            Toast.makeText(LoginActivity.this, "이메일/비밀번호를 확인해 주세요", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
    }
```
<Br>

구글계정으로 로그인 
```java
    private FirebaseAuth firebaseAuth;
    privategooglbtnimage;
    private static final int RC_SIGN_IN = 900;
    private GoogleSignInClient googleSignInClient;
    private SignInButton buttonGoogle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        firebaseAuth = FirebaseAuth.getInstance();
        mDatabase = FirebaseDatabase.getInstance().getReference();
        
        googlbtnimage=(Button)findViewById(R.id.googlbtnimage);
       //구글 로그인 버튼 리스너
        googlbtnimage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                        Intent signInIntent = googleSignInClient.getSignInIntent();
                        startActivityForResult(signInIntent, RC_SIGN_IN);
                        revokeAccess();
            }
        });
        ////사용자 ID정보를 요청(requestemail())하도록 구글 로그인을 구성 하려면 'DEFAULT_SIGN_IN' 매개 변수를 사용하여 googleSignInOptions을 만들었다. 옵션을 선언한 googleSignInOptions으로 googleSignInClient를 선언 합니다.
        GoogleSignInOptions googleSignInOptions = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build();
        googleSignInClient = GoogleSignIn.getClient(this, googleSignInOptions);

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        // 구글로그인 버튼 응답
        if (requestCode == RC_SIGN_IN) {
            Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
            try {
                // 구글 로그인 성공
                GoogleSignInAccount account = task.getResult(ApiException.class);
                firebaseAuthWithGoogle(account);
            } catch (ApiException e) {

            }
        }
    }//사용자가 정상적으로 로그인하면 GoogleSignInAccount 객체에서 ID 토큰을 가져와서 Firebase 사용자 인증 정보로 교환하고 해당 정보를 사용해 Firebase에 인증합니다.인증에 성공하면 MainAcitivity 화면으로 전환 합니다.
    private void firebaseAuthWithGoogle(GoogleSignInAccount acct) {
        AuthCredential credential = GoogleAuthProvider.getCredential(acct.getIdToken(), null);
        firebaseAuth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {                
                            Toast.makeText(LoginActivity.this, R.string.success_login, Toast.LENGTH_SHORT).show();
                            Intent intent=new Intent(getApplicationContext(),MainActivity.class);
                            startActivity(intent);
                            finish();
                        } else {
                            // 로그인 실패
                            Toast.makeText(LoginActivity.this, R.string.failed_login, Toast.LENGTH_SHORT).show();
                        }

                    }
                });
    }
    //구글 로그아웃
    private void revokeAccess() {

        googleSignInClient.revokeAccess()
                .addOnCompleteListener(this, new OnCompleteListener<Void>() {
                    @Override
                    public void onComplete(@NonNull Task<Void> task) {
                        // ...;
                    }
                });
    }
```
<br>

페이스북 계정으로 로그인
```java 
 
//페이스북 로그인을 진행할때 로그인의 응답을 처리할 콜백관리자를 선언한다.
    private CallbackManager callbackManager;
    private LoginStatusCallback loginStatusCallback;
    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        firebaseAuth = FirebaseAuth.getInstance();
        mDatabase = FirebaseDatabase.getInstance().getReference();   
        facebookbtnimage=(Button)findViewById(R.id.facebookbtnimage);
        /페이스북 제공 버튼   로그인 결과에 응답하기위해  loginButton에 콜백을 등록한다.
        LoginButton loginButton = (LoginButton) findViewById(R.id.login_button);
        loginButton = findViewById(R.id.login_button);
        LoginButton finalLoginButton = loginButton;

     
        //커스텀 버튼 리스너
        facebookbtnimage.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v)
        {
            finalLoginButton.performClick();

        }

          });
        //CallbackManager.Factory.create`를 호출하여 로그인 응답을 처리할 콜백 관리자를 만듭니다.
        callbackManager = CallbackManager.Factory.create();
        loginButton.setReadPermissions("email", "public_profile");
        //로그인에 성공하면 `LoginResult` 매개변수에 새로운 `AccessToken`과 최근에 부여되거나 거부된 권한이 포함됩니다
        loginButton.registerCallback(callbackManager, new FacebookCallback<LoginResult>() {
            @Override
            public void onSuccess(LoginResult loginResult) {
                handleFacebookAccessToken(loginResult.getAccessToken());
            }
            @Override
            public void onCancel() {
            }
            @Override
            public void onError(FacebookException error) {
            }
        });
    //사용자가 정상적으로 로그인 했을때  LoginButton의 onSuccess 콜백 매서드에서,로그인한 사용자의액세스 토큰을 가져오 Firebase 사용자 인증 정보로 교환하고 해당 정보를 사용해서 Firebase 인증을 받습니다.
    private void handleFacebookAccessToken( final AccessToken accessToken){
        AuthCredential credential = FacebookAuthProvider.getCredential(accessToken.getToken());
        firebaseAuth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            Toast.makeText(LoginActivity.this, R.string.success_login, Toast.LENGTH_SHORT).show();
                            Intent intent = new Intent(getApplicationContext(), MainActivity.class);
                            startActivity(intent);
                            finish();
                        } else {
                            // 로그인 실패
                            Toast.makeText(LoginActivity.this, R.string.failed_login, Toast.LENGTH_SHORT).show();
                        }
                    }
                });

    }
    //onActivittyResult 에서 callbackManager.onActivityResult를 호출하여 로그인 결과를  LoginManager에 전달 합니다.
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        callbackManager.onActivityResult(requestCode, resultCode, data);
    }
  //페이스북 로그아웃
    private void access(){
        firebaseAuth.getCurrentUser().unlink(PROVIDER_ID)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            // Auth provider unlinked from account
                            // ...
                        }
                    }
                });
    }

}

```
<br>


데이터베이스 저장('Facebook 로그인','Google 로그인' 공통 부분)

```java
    //getInstance를 사용하여 데이터베이스의 인스턴스를 검색하고, 쓰려는 위치를 참조  
    private FirebaseDatabase database = FirebaseDatabase.getInstance();  
    //데이터 베이스에서 데이터를 읽거나 쓰기 위해 DataReference의 인스턴스 선언  
    private DatabaseReference mDatabasef
    ////user는 현재 로그인 되어있는 사용자를 의미한다
    FirebaseUser user = firebaseAuth.getCurrentUser();
    String cu = firebaseAuth.getUid();
    //로그인 사용자의 정보 얻음
    String name = user.getDisplayName();
    String email = user.getEmail();
    //Log 기록 
    Log.v("알림", "현재로그인한 유저 " + cu);
    Log.v("알림", "현재로그인한 이메일 " + email);
    Log.v("알림", "유저 이름 " + name);
    //데이터 저장 확장성을 위하 생성된 userinfo 클래스의 인스턴스 생성
    userinfo userdata = new userinfo(name, email);
    //데이터베이스에서 데이터 쓰기를 위해 DataReference의 인스턴스를 사용. 기본 쓰기 작업은 setValue() 코드를 사용하여 지정된 참조에 데이터를 저장하고 해당 경로의 기존 데이터를 모두 바꿉니다. child는 하위 노드에 작성 하기기위해 사용된다.
    mDatabase.child("users").child(cu).setValue(userdata);
```
<br> 

![아오](https://user-images.githubusercontent.com/62867182/85480553-2d522b80-b5fb-11ea-87bc-4ef20c5358f9.PNG)




<br>

회원가입 버튼 과 비밀번호 찾기 버튼  클릭 리스너 
```java
    //비밀번호찾기 버튼 클릭시 FindActivity 화면으로 전환
    bt_find.setOnClickListener(new View.OnClickListener() {  
    @Override  
       public void onClick(View v) {  
       Intent intent = new Intent(getApplicationContext(), FindActivity.class);  
       startActivity(intent);  
  }  
});
  //회원가입 버튼 클릭시 SignUpActivity 화면으로 전환  
  btn_signup.setOnClickListener(new View.OnClickListener() {  
  @Override  
    public void onClick(View v) {  
    Intent intent = new Intent(getApplication(), SignUpActivity.class);  
    startActivity(intent);  
  }  
    });  
}
```

<br>

 [LoginActivity.java 전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/LoginActivity.java)


<br>

## 비밀번호 찾기
#### FindActivity.java

비밀번호 찾기 기능 사전에 회원가입한 계정의 비밀번호를 분실 하였을 때, 사용하는 기능이다.



```java
    findeamil = (EditText) findViewById(R.id.findemail);  
    but_findpasssword = (Button) findViewById(R.id.but_findpassword);  
    firebaseAuth = FirebaseAuth.getInstance();  
    String eamilAddress=findeamil.getText().toString();  
    //버튼 클릭시 회원가입시 등록하 이메일로 메일을 보내서 재인증함.  
    but_findpasssword.setOnClickListener(new View.OnClickListener() {  
    @Override  
    public void onClick(View v) {  
    String eamilAddress = findeamil.getText().toString().trim();  
    firebaseAuth.sendPasswordResetEmail(eamilAddress).addOnCompleteListener(new OnCompleteListener<Void>() {  
    @Override  
    public void onComplete(@NonNull Task<Void> task) {  
    if (task.isSuccessful()) {  
     Toast.makeText(FindActivity.this, "이메일 보냈습니다.", Toast.LENGTH_SHORT).show();  
     Intent intent=new Intent(getApplicationContext(),LoginActivity.class);  
     startActivity(intent);  
     } else {  
        oast.makeText(FindActivity.this, "이메일 보내기 실패.", Toast.LENGTH_SHORT).show();  
            }  
    }     
        });  
    }  
        });  
    }  
    }
```





[FindActivity.java 전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/FindActivity.java)

<br>

## 회원 가입
<h4>SignUpActivity.java</h4>

회원가입 기능은 구글 계정과 ,페이스북 계정으로 로그인 못하는 상황에서도  사용할 수 있는 기능이다.<br>
이메일과 비밀번호를 작성한 뒤 유효성을 체크하고 오류가 없으면 회원가입이 진행된다. <br>
회원 가입에 성공하면 바로 로그인되며, 메인 화면으로 전환 된다.





```java
    FirebaseAuth firebaseAuth;
    String email = "";
    String password = "";
    EditText ed_singupeamil, ed_signuppassword;
    Button bt_newsignup, bt_backmain;
    TextView tv_error_email,tv_error_password;
    FirebaseDatabase database = FirebaseDatabase.getInstance();
    DatabaseReference mDatabase;// ...
    /이메일 유효성체크 위한 선언
    String emailPattern = "[a-zA-Z0-9._-]+@[a-z]+\\.+[a-z]+";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_signup);
        firebaseAuth = FirebaseAuth.getInstance();
        mDatabase = FirebaseDatabase.getInstance().getReference();
        ed_singupeamil=(EditText)findViewById(R.id.ed_signupemail);
        ed_signuppassword=(EditText)findViewById(R.id.ed_signuppassword);
        bt_newsignup=(Button)findViewById(R.id.bt_newsignup);
        bt_backmain=(Button)findViewById(R.id.bt_backmain);
        tv_error_email = (TextView)findViewById(R.id.tv_error_email);
        tv_error_password=(TextView)findViewById(R.id.tv_error_password);
        //아이디와 비밀번호 작성 후 클릭시 회원가입 진행 후 메인 화면으로 전환
        bt_newsignup.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                email=ed_singupeamil.getText().toString();
                password=ed_signuppassword.getText().toString();
                //이메일 유효성 체크
                if(email.matches(emailPattern))
                {
                    tv_error_email.setText("");         //에러 메세지 제거
                    ed_singupeamil.setBackgroundResource(R.drawable.white_edittext);  //테투리 흰색으로 변경
                }
                else {
                    tv_error_email.setText("이메일 형식으로 입력해주세요.");
                    ed_singupeamil.setBackgroundResource(R.drawable.red_edittext);  // 적색 테두리 적용
                }
                //비밀번호 유효성 체크
                if(password.getBytes().length<6){
                    tv_error_password.setText("비밀번호가 6자리 미만입니다 6자리 이상 입력해주세요.");
                    ed_signuppassword.setBackgroundResource(R.drawable.red_edittext);  // 적색 테두리 적용


                }
                else{
                    tv_error_password.setText("");         //에러 메세지 제거
                    ed_signuppassword.setBackgroundResource(R.drawable.white_edittext);  //테투리 흰색으로 변경

                }
                if(isValidEmail() && isValidPasswd()) {
                    createUser(email, password);
                }

            }
        });
        //로그인 화면으로 전환 
        bt_backmain.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(getApplication(),LoginActivity.class);
                startActivity(intent);
            }
        });
    }
    private boolean isValidEmail() {
        if (email.isEmpty()) {
            // 이메일 공백
            Toast.makeText(SignUpActivity.this,"이메일이 공백입니다.",Toast.LENGTH_SHORT);
            return false;
        }
        else {
            return true;
        }
    }
    // 비밀번호 유효성 검사
    private boolean isValidPasswd() {
        if (password.isEmpty()) {
            // 비밀번호 공백
            Toast.makeText(SignUpActivity.this, "패스워드가 공백입니다.", Toast.LENGTH_SHORT);
            return false;
        } else {
            return true;
        }
    }
    //입력된 eamil과 비밀번호를 가지고 회원가입 진행 
    private void createUser(final String email, final String password) {
        firebaseAuth.createUserWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            // 회원가입 성공
                            Toast.makeText(SignUpActivity.this, "Make You Study에 오신 것을 환영합니다.", Toast.LENGTH_SHORT).show();
                            Intent intent=new Intent(getApplicationContext(),MainActivity.class);
                            startActivity(intent);
                            String cu = firebaseAuth.getUid();
                            userinfo userdata = new userinfo(email, password);
                            mDatabase.child("users").child(cu).setValue(userdata);
                            finish();
                        } else {
                            // 회원가입 실패
                            Toast.makeText(SignUpActivity.this, R.string.failed_signup, Toast.LENGTH_SHORT).show();
                        }
                    }
                });
    }
}
```
<br>

[SignUpActivity.java 전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/SignUpActivity.java)


<br>


## 프로필 (설정)


메인 화면에서 톱니바퀴 모양을 클릭하면 프로필 화면으로 전환 된다 .  
- 책상 이미지 등록하는 기능이다. 
- 로그아웃 기능이다.  
- 회원 탈퇴 기능이다. 

책상 이미지를 등록하기 위해 Firebase Storage를 사용하였다.
Firebase Storage를 사용하기위해 아래와 같이 bulid.gradle (:app)에 추가하여야 한다.
#### bulid.gradle (:app)
~~~java
implementation 'com.google.firebase:firebase-storage:17.0.0'  
implementation 'com.firebaseui:firebase-ui-storage:6.2.1'
~~~
>[Firebase Storage 설명 바로가기 ](https://firebase.google.com/docs/storage/android/start)

![프로필 책상](https://user-images.githubusercontent.com/50138845/85499599-e1b07980-b61c-11ea-9823-4d9dd17bba41.jpg)





### 책상 이미지 등록
사진을 촬영하면 Firebase Storage에 저장 된다. 책상 이미지 등록은 책상 사진을 5장을 찍어서 등록 해야한다.<br>
5장을 모두 등록하면 위의 사진 처럼 빨간불에서 초록불로 변경된다.
Realtime database에는 사진을 처음 등록할때 size와 position값을 초기화 하고 그 이후에 등록할 때는 사진의 개수만큼 count가 증가한다. <br>
또한 position의 값으로 사진을 저장한다.<br>
로그아웃 기능과 회원탈퇴 기능은 버튼을 클릭한뒤 다시 한번 로그아웃과 회원탈퇴를 할 것인지 물어본다.
#### ProfileActivity.java
```java
    private static final String TAG = "MainActivity";
    public static final int UP_COUNT = 1;
    public static final int GET_SIZE = 2;

    FirebaseAuth firebaseAuth;
    private TextView te_textview;
    private Button bt_logout, bt_delect,takeapicture;
    private ImageView imageViewcount;
    //getInstance를 사용하여 데이터베이스의 인스턴스를 검색하고, 쓰려는 위치를 참조 ,데이터 베이스에서 데이터를 읽거나 쓰기 위해 DataReference의 인스턴스 선언  
    DatabaseReference mdatabase = FirebaseDatabase.getInstance().getReference();
    static final int REQUEST_IMAGE_CAPTURE = 1;

    //firebase에 자신의 책상 image 등록해주기 위함 ,스토리지 버킷에 액세스하는 첫 단계는 FirebaseStorage의 인스턴스 todtjd
    FirebaseStorage storage = FirebaseStorage.getInstance();
    //참조(파일 업로드, 다운로드, 삭제, 메타데이터 가져오기 또는 업데이트를 하려면 필요)를 만들려면 FirebaseStorage 싱글톤 인스턴스를 사용하고 이 인스턴스의 getReference() 메서드를 호출
    StorageReference storageRef;
    private FirebaseUser user;
    int position;
    int size;
    android.app.AlertDialog waitingDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_profile);

        te_textview = (TextView) findViewById(R.id.te_textview);
        bt_logout = (Button) findViewById(R.id.bt_logut);
        bt_delect = (Button) findViewById(R.id.bt_delect);
        imageViewcount = (ImageView)findViewById(R.id.imageViewCount);

        takeapicture=(Button)findViewById(R.id.takeapicture);
        firebaseAuth = FirebaseAuth.getInstance();
        user = firebaseAuth.getCurrentUser();
        te_textview.setText(user.getEmail() + "으로 로그인 하였습니다.");
        FirebaseDatabase database = FirebaseDatabase.getInstance();
        final DatabaseReference mDatabase;
        String cu = firebaseAuth.getUid();
         //storage 에 저장 되는 형식 설정 
        storageRef = storage.getReference().child("images").child(user.getUid());
        checksize(GET_SIZE);

        waitingDialog = new SpotsDialog.Builder().
                setContext(this)
                .setMessage("Please waiting...")
                .setCancelable(false).build();

        //사진 찍기 리스너너
       takeapicture.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dispatchTakePictureIntent();
            }
        });
    }

    // 사진 찍기
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);

    }

    @Override //사진촬영후 사진 저장
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // Check which request we're responding to
        super.onActivityResult(requestCode, resultCode, data);
        checksize(GET_SIZE);
        if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == Activity.RESULT_OK && data.hasExtra("data")) {
            Bitmap bitmap = (Bitmap) data.getExtras().get("data");
            if (bitmap != null) {
                checksize(UP_COUNT);
                imageUpload(bitmap);
            }
        }
    }
    //firebase에 책상 image upload method, `putBytes()`는 `byte[]`를 취하고 `UploadTask`를 반환하며 이 반환 객체를 사용하여 업로드를 관리하고 상태를 모니터링할 수 있습니다.
    public void imageUpload(Bitmap bmpImage){
        waitingDialog.show();

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        bmpImage.compress(Bitmap.CompressFormat.JPEG, 100, baos);
        byte[] data1 = baos.toByteArray();

        StorageReference filepath = storageRef.child(position+"");

        UploadTask uploadTask = filepath.putBytes(data1);
        uploadTask.addOnFailureListener(new OnFailureListener() {
            @Override
            public void onFailure(@NonNull Exception exception) {

                if(waitingDialog.isShowing())
                    waitingDialog.dismiss();

                Toast.makeText(ProfileActivity.this, "이미지 등록에 실패하였습니다.", Toast.LENGTH_SHORT).show();
            }
        }).addOnSuccessListener(new OnSuccessListener<UploadTask.TaskSnapshot>() {
            @Override
            public void onSuccess(UploadTask.TaskSnapshot taskSnapshot) {

                if(waitingDialog.isShowing())
                    waitingDialog.dismiss();

                Log.d("Upload : ", "Success");
                Toast.makeText(ProfileActivity.this, "이미지가 성공적으로 등록되었습니다.", Toast.LENGTH_SHORT).show();
                Log.d("TEST", "SIZE : " + size + "POSITION : " + position);
                mdatabase.child("image").child(user.getUid()).child("size").setValue(size+"");
                mdatabase.child("image").child(user.getUid()).child("position").setValue(position+"");
            }
        });
    }


    // image database가 null인지 확인 후 null이면 초기화
    public void checksize(int mode){
                mdatabase.child("image").child(user.getUid()).addListenerForSingleValueEvent(new ValueEventListener() {
                    @Override
                    public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                        Log.d(TAG, "CHECKSIZE가 실행");
                        if(dataSnapshot.getValue() == null){
                            Log.d("Checksize : ", "Uid child is null");
                            // 처음 등록할 때 size값과 position값을 초기화시켜준다.
                            mdatabase.child("image").child(user.getUid()).child("size").setValue("0");
                            mdatabase.child("image").child(user.getUid()).child("position").setValue("0");
                            size = 0;
                            position = 0;
                        }else{
                            size = Integer.parseInt(dataSnapshot.child("size").getValue(String.class));
                            position = Integer.parseInt(dataSnapshot.child("position").getValue(String.class));

                        }
                        if(mode == UP_COUNT){
                            countPosition();
                        }
                        if(size > 4){
                            imageViewcount.setImageDrawable(getDrawable(R.drawable.ic_green));
                        }
                    }
            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) { }
        });
    }
    // user별 database에 저장된 현재 position값 계산
    public void countPosition(){
        if(size < 5){ size++; }
        if(position > 3){ position = 0; } else{ position++; }
    }
}

```
[Firebase Storage Upload 설명 바로가기](https://firebase.google.com/docs/storage/android/upload-files)

<br>

### 로그아웃 


```java  

        // 로그아웃 버튼 리스너
        bt_logout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                logoutDialog();
            }
        });
        //리스너 클릭시, 다시 한번 할것인지 물어보고 응답에 따라  진행
    public void logoutDialog(){
        AlertDialog.Builder bui=new AlertDialog.Builder(this);
        bui.setTitle("로그아웃");
        bui.setMessage("로그아웃 하시겠습니까?");
        bui.setPositiveButton("예", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
               //사용자 로그아웃
                FirebaseAuth.getInstance().signOut()ㅣ
                firebaseAuth.signOut();
                //페이스북 로그아웃
                LoginManager.getInstance().logOut();
                finish();
                Intent intent = new Intent(getApplicationContext(), LoginActivity.class);
                startActivity(intent);
                finish();
            }
        }).setNegativeButton("아니오", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                Intent intent = new Intent(getApplicationContext(), ProfileActivity.class);
                startActivity(intent);
            }
        });
        bui.show();
    }

```
<BR>

### 회원 탈퇴

```java
        //회원탈퇴 리스너
        bt_delect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Dialog();
            }
        });

});
//리스너 클릭시, 다시 한번 탈퇴할것인지 물어보고 응답에 따라  진행
public void Dialog () {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("회원 탈퇴");
        builder.setMessage("탈퇴 하시겠습니까?");
        builder.setPositiveButton("예", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                FirebaseUser user = FirebaseAuth.getInstance().getCurrentUser();

                user.delete()
                        .addOnCompleteListener(new OnCompleteListener<Void>() {
                            @Override
                            public void onComplete(@NonNull Task<Void> task) {
                                if (task.isSuccessful()) {
                                    Toast.makeText(ProfileActivity.this, "계정이 삭제 되었습니다.", Toast.LENGTH_LONG).show();
                                    firebaseAuth.getInstance().signOut();
                                    Intent intent = new Intent(getApplicationContext(), LoginActivity.class);
                                    startActivity(intent);
                                }
                            }
                        });
                String cu = firebaseAuth.getUid();
                mdatabase.child("users").child(cu).setValue(null);
            }
        });
```
<br>

[ProfileActivity.java 전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/ProfileActivity.java)

<br>

#### userinfo.java

 한개의 데이터가 아닌 여러개의 데이터 저장을 위해서 , 클래스는 선언 하였다.
```java 
public class userinfo {
    private String userName;
    private String profile;
    private String usercal;
    private String usertable;
    private String usercheck;
    private String userdiary;
    public userinfo(String userName, String profile) {
        this.userName = userName;
        this.profile = profile;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getProfile() {
        return profile;
    }
    public void setProfile(String profile) {
        this.profile = profile;

    }


}

```

[userinfo.java 코드 전체 보기 ](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/userinfo.java)

##   Calendar
Calendar기능은 자신의 한달 일정을 확인 할 수 있고 일정에 따라 공부계획 수정이 가능하도록 도와준다.

-  달력의 커스텀
  -상단에 현재 달이 표시되고 생 오늘날짜는 숫자가 베이지 색, 선택날짜는 원으로 보여준다.
- 자신의 일정 추가하거나 수정한다.
- 일정이 저장이 되면 Calendar에 새싹모양(dot)으로 표시된다.

![calendar custom](https://user-images.githubusercontent.com/62635984/85451749-2c0e0800-b5d5-11ea-8ccb-5ce39652ea10.JPG)

#### activity_Calendar.xml 

 Custom Calendar를 만들기위해 materialcalendarview를  가져온다.

~~~java
<com.applandeo.materialcalendarview.CalendarView  
  android:id="@+id/calendarView"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  app:type="one_day_picker"  
  app:headerColor="@color/mainColor"  
  app:selectionColor="@color/colorPrimaryDark"  
  app:todayLabelColor="@color/colorPrimaryDark"  
  app:eventsEnabled="true"/>
~~~
<br>

 build.gradle에 meterialCalendar를  추가한다.
~~~java
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
~~~
~~~java
dependencies {
    implementation 'com.applandeo:material-calendar-view:1.7.0'
}    
~~~
###  
<br>

#### CalendarActivity.java

 변수를 선언한다. 

~~~java
// 선태한 날짜의 일정을 쓰거나 기존에 저자된 일기가 있다면 보여주고 수정하는 영역  
EditText edtDiary;// 선태한 날짜의 일정을 쓰거나 기존에 저자된 일기가 있다면 보여주고 수정하는 영역
Button btnSave; //선택된 날짜의 파일이름  
  
// 선택한 날짜  
int checkYear;  
int checkMonth;  
int checkDay;
~~~
<br>

  뷰에 있는  위젯들을 리턴 받는다.
~~~java
edtDiary=(EditText)findViewById(R.id.edtDairy);  
btnSave=(Button)findViewById(R.id.btnSave);  
CalendarView calendarView = (CalendarView) findViewById(R.id.calendarView);  
List<EventDay> events = new ArrayList<>();  
  ~~~
  <br>
  
  오늘 날짜와 현재 선택한 날짜를 받는다.
  ~~~java
// 오늘 날짜 받게하기  
Calendar today= Calendar.getInstance();  
int todayYear=today.get(Calendar.YEAR);  
int todayMonth=today.get(Calendar.MONTH);  
int todayDay=today.get(Calendar.DAY_OF_MONTH);  
  
// 현재 선택한 날짜  
checkYear = todayYear;  
checkMonth = todayMonth;  
checkDay = todayDay;
~~~
<br>

Calendar를 처음 시작할 때 일정이 있으면 ic_sprout(새싹모양)으로 표시해준다.
~~~java
 // 첫시작 할 때 오늘 날짜 일정 읽어주기
checkedDay(todayYear, todayMonth, todayDay);

// 일정데이터가 변경될 때 onDataChange함수 발생 한다.
mFirebaseDatabase.getReference().child("calendar").child(user.getUid()).addValueEventListener(new ValueEventListener () {

    @Override
    public void onDataChange(@NonNull DataSnapshot dataSnapshot) { 
        for (DataSnapshot snapshot : dataSnapshot.getChildren()){
            String key = snapshot.getKey();
            int[] date = splitDate(key);
            Calendar event_calendar = Calendar.getInstance();
            event_calendar.set(date[0], date[1], date[2]);
            EventDay event = new EventDay(event_calendar, R.drawable.ic_sprout);
            events.add(event);
        }
        calendarView.setEvents(events);
    }
    
    @Override
    public void onCancelled(@NonNull DatabaseError databaseError) { }
});
~~~
<br>

선택 날짜가 변경될 때 set OnDayClickListenr가 호출된다.

~~~java
calendarView.setOnDayClickListener(new OnDayClickListener () {
    @Override
    public void onDayClick(EventDay eventDay) {
        Calendar clickedDayCalendar = eventDay.getCalendar();
        //이미 선택한 날짜에 일기가 있는지 없는지 체크
        checkedDay(clickedDayCalendar.get(Calendar.YEAR),
                clickedDayCalendar.get(Calendar.MONTH),
                clickedDayCalendar.get(Calendar.DATE));
        //체크한 날짜 변경
        checkYear = clickedDayCalendar.get(Calendar.YEAR);
        checkMonth = clickedDayCalendar.get(Calendar.MONTH);
        checkDay = clickedDayCalendar.get(Calendar.DATE);
    }
});
~~~
<br>

새 일정 추가하거나 수정하는 버튼을 누르면 setOnClickListener가 호출된다.
~~~java
btnSave.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        //fileName을 넣고 저장시키는 메소드를 호출
        saveDiary(checkYear + "-" + checkMonth + "-" + checkDay);
    }
});
~~~
<br>

날짜를 선택하면 checkedDay 함수가 호출되고 일정 Database를 읽는다.
~~~java
private void checkedDay(int year, int monthOfYear, int dayOfMonth) {
    
    // mDatabaseReference의 경로를 filebase/diary/userUid/date 로 설정
    mDatabaseReference = mFirebaseDatabase.getReference().child("calendar").child(user.getUid()).child(year + "-" + monthOfYear + "-" + dayOfMonth);
    mDatabaseReference.addValueEventListener(new ValueEventListener() {
    
        @Override
        public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
            String str = dataSnapshot.getValue(String.class);
            if(str==null){
                // 데이터가 없으면 일정이 없는 것 -> 일정을 쓰게 하기
                edtDiary.setText("");
                //btnSave.setText("새 일정 추가");
                btnSave.setBackgroundResource(R.drawable.ic_newsave);
            }else{
                // mDatabaseReference 경로에 저장된 str을 받아온다.
                edtDiary.setText(str);
                //btnSave.setText("수정하기");
                btnSave.setBackgroundResource(R.drawable.ic_fix);
            }
        }
        
        @Override
        public void onCancelled(@NonNull DatabaseError databaseError) {  }
    });
}
~~~
<br>

일정을 저장할때 saveDiary 메소드가 호출된다.
~~~java
@SuppressLint("WrongConstant")
private void saveDiary(String readDay){
    try{ //일정이 저장될때 try문 발생.
        mDatabaseReference = mFirebaseDatabase.getReference().child("calendar").child(user.getUid()).child(readDay);
        String content =edtDiary.getText().toString();
        // filebase/calendar/userUid/date save
        mDatabaseReference.setValue(content);
        //일정이 저장되면 토스메세지로 "일정 저장 됨"
        Toast.makeText(getApplicationContext(),"일정 저장 완료",Toast.LENGTH_SHORT).show();
    } catch (Exception e){             //예외처리.
        e.printStackTrace();
        Toast.makeText(getApplicationContext(),"오류발생",Toast.LENGTH_SHORT).show();
    }
}
~~~
<br>

문자열을 int로 변환한다.
~~~java
//문자열을 int로 변환한다.
private int[] splitDate(String date){
    String[] splitText = date.split("-");
    int[] result_date = {Integer.parseInt(splitText[0]), Integer.parseInt(splitText[1]), Integer.parseInt(splitText[2])};
    return result_date;
}
~~~

>[CalendarActivity.java 전체 코드 보기](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/CalendarActivity.java)
## Timetable

사용자가 시간표를 추가하여 설정된 시간에 진동과 벨소리가 작동되며 알람이 울리도록하는 기능이다.
- 시간표를 추가하여 새로운 알람을 등록한다.
- 알람이 설정된 시간이되면 AlarmReceiver가 호출되고 AlarmService를 실행한다.
- 알람을 종료하기 위해 다시 AlarmReceiver를 호출하여 AlarmService를 정지시킨다.


![타타임테이블](https://user-images.githubusercontent.com/62635984/85487805-3ac1e280-b608-11ea-922c-d3fa24ae73e8.png)

TimeTable은 TimeTableView, TimeTableActivity, EditActivity, AlarmReceiver, AlarmService 로 나눠서 설명할 것이다.

### TimeTableView
TimeTableView를 사용하기에 앞서 build.gradle(Module:**project**) 파일에 다음을 추가한다.
~~~java
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io'}
    }
}
~~~
> github에서 프로젝트를 Clon or Download 했을 때는 직접 ProjectView를 Project로 변경하면 수정할 수 있습니다.
> ![Projectmodule2](https://user-images.githubusercontent.com/46085058/85412389-7cb93d00-b5a4-11ea-95f1-0d0cf25c5061.png) 

build.gradle(Module:**app**) dependency에 다음을 추가한다.
~~~java
dependencies {
    implementation 'com.github.tlaabs:TimetableView:1.0.3-fx1'
}
~~~
**TimeTableView in Layout.xml**
~~~java
<com.github.tlaabs.timetableview.TimetableView  
    android:id="@+id/timetable"  
    android:layout_width="match_parent"  
    android:layout_height="wrap_content"  
    app:column_count="8"  // 일주일을 표시 
    app:row_count="25"  // 24시를 표시
    app:start_time="0"  // 시작시간을 0으로 표시
    app:header_title="@array/my_header_title"  // header title(요일)을 values/strings.xml에서 받아온다.
    app:header_highlight_color="@color/mainColor"  // 사용자가 별도로 지정한 header색상 변경(ex : 오늘의 요일)
    app:header_highlight_type="color"  // color 또는 image를 지정
  />
~~~
> 더 많은 [TimeTable layout 설정](https://github.com/tlaabs/TimetableView#attribute-descriptions)

 app:header_title 변경
TimeTableView의 header속성을 변경할 수 있습니다.
~~~java
<string-array name="my_header_title">  
   <item></item> <item>Mon</item>  
   <item>Tue</item>  
   <item>Wed</item>  
   <item>Thu</item>  
   <item>Fri</item>  
   <item>Sat</item>  
   <item>Sun</item>  
</string-array>
~~~
> TimeTableViewLayout.xml 
> ~~~java
> app:row_count = "8" // row_count가 item 갯수 보다 1 더 커야 합니다. 
> ~~~

**TimeTableActivity.Java**
<br>TimeTableActivity의 주요 기능은 시간표를 표시하고 알람을 등록해주는 역할을 수행한다.
기존 AlarmManger에 등록된 알람과의 충돌을 방지하기 위해서 기존의 알람을 모두 삭제한 후 재등록 한다.
~~~java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_timetable);
    // init firebase
    ...
    // AlarmManger Service
    alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
    
    init(); // TimeTableView의 view 객체 및 listener 선언
    checkPictureCount(); // 이미지매칭용 사진이 저장되어있는지 확인
    dayCheckZero(); // TimeTable을 처음 사용할 때 출석 값 0으로 초기화
    
    // addValueEventListener를 선언하여 시간표가 수정될 때마다 Alarm을 갱신 해줍니다.
    mDatabaseReference.addValueEventListener(new ValueEventListener() {
        @Override
        public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
            if(dataSnapshot.child("count").getValue(Integer.class) != null){
                count = dataSnapshot.child("count").getValue(Integer.class);
                alarmOff(count);
            }
            if(dataSnapshot.child("table").getValue(String.class) != null){
                timetable.load(dataSnapshot.child("table").getValue(String.class));
                AddAlarm(dataSnapshot.child("table").getValue(String.class));
            }
        }
        @Override
        public void onCancelled(@NonNull DatabaseError databaseError) { }
    });
}
~~~
알람을 등록, 수정, 삭제 기능을 수행하기 위해서 EditActivity에서 받아온 data를 활용한다.
받아온 data는 timetable.createSaveData() 를 활용하여 Json형식으로 저장할 수 있습니다.
- Json형식으로 바뀐 data를 Database에 저장한다.
~~~java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    switch (requestCode) {
        case REQUEST_ADD:
            if (resultCode == EditActivity.RESULT_OK_ADD) {
                ArrayList<Schedule> item = (ArrayList<Schedule>) data.getSerializableExtra("schedules");
                timetable.add(item); // 받아온 Schedule List을 Json 형식으로 저장합니다.
                mDatabaseReference.child("table").setValue(timetable.createSaveData()); // Database에 Json형식으로 저장합니다.
                Toast.makeText(this, "시간표가 추가 되었습니다.", Toast.LENGTH_SHORT).show();
            }
            break;
        case REQUEST_EDIT:
            /** Edit -> Submit */
            if (resultCode == EditActivity.RESULT_OK_EDIT) {
                int idx = data.getIntExtra("idx", -1);
                ArrayList<Schedule> item = (ArrayList<Schedule>) data.getSerializableExtra("schedules");
                timetable.edit(idx, item); // 받아온 data로 idx의 Schedule을 수정합니다.
                mDatabaseReference.child("table").setValue(timetable.createSaveData()); // Database에 Json형식으로 저장합니다.
                Toast.makeText(this, "시간표가 수정 되었습니다.", Toast.LENGTH_SHORT).show();
            }
            /** Edit -> Delete */
            else if (resultCode == EditActivity.RESULT_OK_DELETE) {
                int idx = data.getIntExtra("idx", -1);
                timetable.remove(idx); // 특정 idx의 Schedule을 삭제합니다.
                mDatabaseReference.child("table").setValue(timetable.createSaveData()); // Database에 Json형식으로 저장합니다.
                Toast.makeText(this, "시간표가 삭제 되었습니다.", Toast.LENGTH_SHORT).show();
            }
            break;
        }
    }
~~~
> Json형식
> 
> ![Json형식](https://user-images.githubusercontent.com/46085058/85429692-12ab9280-b5ba-11ea-882b-b958e299604f.PNG)

#### Method
출석률을 초기화해주는 메서드 [`dayCheckZero()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java#L337)
~~~java
public void dayCheckZero(){...}
~~~
출석체크의 이미지매칭을 위한 등록된 사진갯수를 확인하는 메서드 [`checkPictureCount()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java#L299)

~~~java
public void checkPictureCount(){...}
~~~
JsonParsing 후 알람을 등록하는 메서드 ( 알람의 등록과 삭제는 따로 다루도록 한다.) [`AddAlarm()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java#L212)
~~~java
public void AddAlarm(String json){...}
~~~
> JsonParse 참고 : [https://jang8584.tistory.com/185](https://jang8584.tistory.com/185)
> 

알람을 삭제하는 메서드 ( 알람의 등록과 삭제는 따로 다루도록 한다.) [`alarmOff()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java#L354)
~~~java
public void alarmOff(int tmpcount){...}
~~~
#### Alarm등록 및 삭제 
Alarm의 등록 및 삭제는 중요한 몇 가지가 기능을 좌우하기 때문에 별도로 설명하도록 한다.
> AlarmReceiver.class가 있다는 가정하에 설명하겠다.

알람의 object 선언
~~~java
//Alarm object  
private AlarmManager alarmManager;  
private PendingIntent pendingIntent;
~~~
> PendingIntent은 별도의 컴포넌트에게 내가 보내고자하는 intent를 대신 전달하고자 할 때 선언한다.
> 주로 외부에서 Activity, Broadcast, Service를 사용해야 할 때 선언한다.
> 여기에서는 Broadcast 컴포넌트로 다뤄보도록 하겠다.
> 
Pendingintent 설정
~~~java
Intent intent = new Intent(getApplicationContext(), AlarmReceiver.class);  
intent.putExtra("weekday", obj3.get("day").getAsInt());  
intent.putExtra("state", "on"); // state 값이 on 이면 알람시작, off 이면 중지, day는 Receiver에서 구분  
intent.putExtra("reqCode", i);  
pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), i, intent, PendingIntent.FLAG_CANCEL_CURRENT);
~~~
> intent에 담아야할 정보들을 Intent.putExtra() 로 담아주고, Broadcast 컴포넌트를 받아온다.
> 이 때 **FLAG**에 주목해야 하는데 FLAG는 Pendingintent의 충돌을 막아주는 아주 중요한 역할을 수행한다.
> - FLAG_CANCEL_UPDATE : 이전에 생성된 pendingintent가 있으면 삭제하고 새롭게 생성한다.
> - FLAG_UPDATE_CURRENT : 이전에 생성된 pedingintent의 내용을 갱신 한다.
> - FLAG_ONE_SHOT : 일회용으로 생성 pendingintent를 단 1회만 사용한다. ( 등록한 위젯이 있다면 1회 클릭 후에는 반응하지 않는다. )
> - FLAG_NO_CREATE : 이미 생성된 것이 있다면 삭제한다.
> 
> Make You Study에서는 알람시간 변경 및 추가가 계속 이루어져야 함으로 FLAG_CANCEL_UPDATE 또는 FLAG_UPDATE_CURRENT를 사용한다.

PendingIntent를 구분하는 **RequestCode** 알람을 판단하는 가장 중요한 부분이라고 할 수 있다.
출석체크를 완료할 때 어떤 알람이 실행되었는지를 판단하고 해당 RequestCode번호의 알람을 삭제하는 역할을 수행하기 때문에 **중복을 피한다**.
~~~java
pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), requestcode, intent, PendingIntent.FLAG_CANCEL_CURRENT);
~~~

AlarmManger 설정
AlarmManger는 Pendingintent를 어느 시점에 실행할지 결정해주는 역할을 수행한다.
~~~java
Calendar calendar = Calendar.getInstance(); // Calendar 객체 생성
calendar.set(Calendar.HOUR_OF_DAY, obj4.get("hour").getAsInt()); // 시간 설정
calendar.set(Calendar.MINUTE, obj4.get("minute").getAsInt());  // 분 설정
calendar.set(Calendar.SECOND, 0); // 초 설정(되도록이면 Default로 0을 둔다.)
~~~
> obj4.get("...").getAsInt() 는 MakeYouStudy의 시간을 불러오는 [예제](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java#L250)이다. 원하는 시간대로 설정해주면 된다.
> 
시간을 설정했으면 알람을 등록 해주어야 한다.  이때 **API**마다 실행 방식이 다르기 때문에 꼭 아래와 같이 작성해 주어야 한다.
~~~java
if(Build.VERSION.SDK_INT < Build.VERSION_CODES.M){
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
        // API 19이상 API 23미만
        alarmManager.setExact(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
    }else{
        // API 19미만
        alarmManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
    }
}else{
    // API 23이상
    alarmManager.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
}
~~~

[TimeTableActivity.java 전체 코드]([https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/TimeTableActivity.java))

**EditActivity.Java**
<br>EditActivity의 주요기능은 시간표를 생성, 수정 및 삭제하는 역할을 수행한다.

EditActivity는 시간표 생성과 수정을 구분하여야 한다. 특히 수정시에는 이미 등록되어있는 시간표의 정보를 불러오는 기능을 수행한다.
~~~java
private void checkMode(){
        Intent i = getIntent();
        mode = i.getIntExtra("mode",TimeTableActivity.REQUEST_ADD);

        if(mode == TimeTableActivity.REQUEST_EDIT){
            loadScheduleData(RESULT_OK_EDIT);
            deleteBtn.setVisibility(View.VISIBLE);
        }else if(mode == TimeTableActivity.REQUEST_ADD){
            loadScheduleData(0);
        }
    }
~~~
EDIT 모드일 때는 받아온 시간표로 View들을 업데이트 시켜주고, ADD모드에서는 현재시간을 등록하여 사용자가 쉽게 시간을 선택할 수 있도록 도와준다.
~~~java
private void loadScheduleData(int mode){
    if(mode == RESULT_OK_EDIT){
        Intent i = getIntent();
        editIdx = i.getIntExtra("idx",-1);
        ArrayList<Schedule> schedules = (ArrayList<Schedule>)i.getSerializableExtra("schedules");
        schedule = schedules.get(0);
        subjectEdit.setText(schedule.getClassTitle());
        classroomEdit.setText(schedule.getClassPlace());
        professorEdit.setText(schedule.getProfessorName());
    }
    daySpinner.setSelection(schedule.getDay());
    startTv.setText(schedule.getStartTime().getHour()+":"+schedule.getStartTime().getMinute());
    endTv.setText(schedule.getEndTime().getHour()+":"+schedule.getEndTime().getMinute());
}
~~~
설정한 시간을 schedule에 담아준다.
~~~java
private void inputDataProcessing(){
    schedule.setClassTitle(subjectEdit.getText().toString());
    schedule.setClassPlace(classroomEdit.getText().toString());
    schedule.setProfessorName(professorEdit.getText().toString());
}
~~~
>  [Schedule.set...()](https://github.com/tlaabs/TimetableView#add-schdule)
#### Method
EditActivity View object 초기화[`init()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/EditActivity.java#L56)
~~~java
public void init(){...}
~~~
EditActivity View Listener 초기화[`initView()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/EditActivity.java#L88)
~~~java
private void initView(){...}
~~~
TimeTableActivity로 생성, 수정 및 삭제를 구별하여 intent를 전송하는 [`onClick()`](https://github.com/JJinTae/MakeYouStudy/blob/d985189ef614f284db09d27648f8d6abfebd491f/app/src/main/java/com/android/MakeYouStudy/EditActivity.java#L137)
~~~java
@Override public void onClick(View v) {...}
~~~

[EditActivity.java 전체 코드]([https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/EditActivity.java](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/EditActivity.java))

**AlarmReceiver.Java**
<br>AlarmReceiver는 Alarm Broadcast Message를 수신하는 역할을 수행한다.

**AlarmReceiver.Java 생성**
<br>AlarmReceiver는 Broadcast를 수신하기 위해서 새로운 Java파일을 생성할 때 BroadcastReceiver를 extends해야 한다.
![Creat_AlarmReceiver](https://user-images.githubusercontent.com/46085058/85443237-fb75a080-b5cb-11ea-8766-094b58e3bd87.png)
![extends_AlarmReceiver](https://user-images.githubusercontent.com/46085058/85443431-38da2e00-b5cc-11ea-98a4-b99fe66d631a.png)
BroadcastReceiver를 extends하였기 때문에 onReceive() 를 선언 해주어야한다.
![onReceiver_AlarmReceiver](https://user-images.githubusercontent.com/46085058/85444017-dcc3d980-b5cc-11ea-8a02-d19132dd9307.png)
intent의 state값을 받아오고 "off", "on"은 알람을 끄거나 켤 때 사용하는 state이고  "reset"은  media를 조절하기 위해 별도로 만든 state이다.
weeks는 해당 요일을 구별하여 오늘의 요일이 맞지않으면 알람을 실행하지 않는다.
각 state를 구별한 후에 Service를 호출한다. (Android Oreo 이상 부터는 foreground로 실행하여야 한다.)
~~~java
public class AlarmReceiver extends BroadcastReceiver {
    static String TAG="AlarmReceiver";

    ...PowerManger

    @Override
    public void onReceive(Context context, Intent intent) {
        ...
        // intent에 담겨져있는 값들을 받아온다.
        int weeks = intent.getIntExtra("weekday", -1);
        int reqCode = intent.getIntExtra("reqCode", -1);
        String state = intent.getStringExtra("state");
        
        Intent sIntent = new Intent(context, AlarmService.class); // Service로 보낼 intent

        if(state.equals("off")){ // 알람을 종료하는 state
            sIntent.putExtra("state", "off");
            sIntent.putExtra("reqCode", reqCode);
            // Oreo(26) 버전 이후부터는 Background 에서 실행을 금지하기 때문에 Foreground 에서 실행해야 함
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(sIntent);
            } else {
                context.startService(sIntent);
            }
            return;
        }
        else if(state.equals("reset")){ // Service의 Media컨트롤을 위한 state
            Log.d(TAG, "reset " + reqCode + " 가 해제 되었습니다.");
        }
        else if(weeks != nweeks){ // 오늘이 설정한 요일이 아닐 때 아무것도 수행하지 않음
            return;
        }
        else if(weeks == nweeks) { // 오늘이 설정한 요일 일 때 알람이 울림
            sIntent.putExtra("state", "on");
            sIntent.putExtra("weekday", weeks);
            // Oreo(26) 버전 이후부터는 Background 에서 실행을 금지하기 때문에 Foreground 에서 실행해야 함
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(sIntent);
            } else {
                context.startService(sIntent);
            }
            
            ...PowerManger
            
            try { // 오늘이 설정한 요일일 때 출석체크 액티비티를 실행해준다.
                Intent intent2 = new Intent(context, AttendanceCheckActivity.class);
                intent2.putExtra("reqCode", reqCode);
                intent2.putExtra("weekday", weeks);
                PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent2, PendingIntent.FLAG_CANCEL_CURRENT);
                pendingIntent.send();

            } catch (PendingIntent.CanceledException e) {
                e.printStackTrace();
            }
            
            ...PowerManger
            
            }
        }
    }
}
~~~
알람이 해당하는 요일과 일치하여 알람이 울려야할 때 사용자의 화면을 깨워준다.
이는 절전모드 상태에서 CPU자원을 획득하기 때문에 필히 Release 해주어야한다.

~~~java
// PowerManger.WakeLock object
private static PowerManager.WakeLock sCpuWakeLock;  
private static ConnectivityManager manger;

@Override
public void onReceive(Context context, Intent intent) {
        ...Alarm
        // 절전모드에서도 액티비티를 띄울 수 있도록 함
        PowerManager pm = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
        sCpuWakeLock = pm.newWakeLock(PowerManager.SCREEN_BRIGHT_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.ON_AFTER_RELEASE, "app:alarm");
        // acquire 함수를 실행하여 앱을 깨운다. (CPU를 획득함)
        sCpuWakeLock.acquire();
        manger = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        ...Alarm
        // acquire 함수를 사용하였으면 꼭 release를 해주어야 한다.
        // cpu를 점유하게 되어 배터리 소모나 메모리 소모에 영향을 미칠 수 있다.
        if (sCpuWakeLock != null) {
            sCpuWakeLock.release();
            sCpuWakeLock = null;
        }
    }
}
~~~

[AlarmReceiver 전체 코드]([https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AlarmReceiver.java](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AlarmReceiver.java))

###  AlarmService.Java
정해진 시간에 알람이 울렸을 때 Service를 통하여 Vibrator와 Media를 재생할 수 있도록 해준다.

**AlarmService.java 생성**
<br>AlarmService는 새로운 Java파일을 생성할 때 Service를 extends해야 한다.
![Create_Service](https://user-images.githubusercontent.com/46085058/85454120-8d36db00-b5d7-11ea-80e9-3dc31e2bb292.png)

처음 Service를 extends한 Java파일을 생성하게되면 오류가 뜨는데 아래와 같이 `onBind()`와 `onStartCommand()` 를 선언해주어야 한다.
~~~java
public class AlarmService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {return null;}
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }
}
~~~

`onStartCommand()`는 Service가 호출되었을 때 실행된다.
AlarmService에서는 주로 `MediaPlayer()`의 재생, 중지 및 정지와, `Vibrate()`의 재생 및 정지를 수행하고 추가로 Android Oreo버전 이상부터는 foreground실행을 위해 notificationchannel을 띄워주는 역할을 수행한다.
~~~java
public class AlarmService extends Service {

    private MediaPlayer mediaPlayer;
    private Vibrator vibrator;
    private boolean isRunning;
    private int pausePosition; // mediaPlayer pause 시점 저장

    @Nullable
    @Override
    public IBinder onBind(Intent intent) { return null; }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        vibrator = (Vibrator)getSystemService(Context.VIBRATOR_SERVICE); // System에서 Vibrater Service를 받아온다.
      // intent값을 받아온다.
        String state = intent.getStringExtra("state");
        int reqCode = intent.getIntExtra("reqCode", -1);
        int weeks = intent.getIntExtra("weekday", -1);

        if (state.equals("on")) {
            // 알람음 재생 OFF, 알람음 시작 상태
            this.mediaPlayer = MediaPlayer.create(this, R.raw.alarm); // 재생할 음악을 정한다.
            this.mediaPlayer.start(); // 음악을 재생
            this.vibrator.vibrate(new long[]{500, 1000, 500, 1000}, 0); // 진동 재생

            this.isRunning = true;

            // notification 클릭시에도 출석체크를 할 수 있도록 액티비티를 실행
            Intent intent1 = new Intent(getApplicationContext(), AttendanceCheckActivity.class);
            intent1.putExtra("reqCode", reqCode);
            intent1.putExtra("weekday", weeks);
            PendingIntent pendingIntent = PendingIntent.getActivity(getApplicationContext(), 0, intent1, PendingIntent.FLAG_CANCEL_CURRENT);

            // Oreo(26) 버전 이후 버전부터는 notificationchannel 이 필요함
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                
                String channelId =  createNotificationChannel();
                NotificationCompat.Builder builder = new NotificationCompat.Builder(this, channelId);
                Notification notification = builder.setOngoing(true)
                        .setSmallIcon(R.mipmap.ic_launcher) // 아이콘을 설정 
                        .setContentIntent(pendingIntent) 
                        .build();
                startForeground(1, notification);
            }
        } else if (this.isRunning && state.equals("off")) {
            // 알람음 재생 ON, 알람음 중지 상태
            this.mediaPlayer.stop(); // 음악을 정지
            this.mediaPlayer.reset(); // mediaPlayer를 리셋
            this.mediaPlayer.release(); // mediaPlayer 해제
            this.vibrator.cancel(); // 진동 정지

            this.isRunning = false;

            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                stopForeground(true); // 실행중인 foreground 정지
            }
        } else if (state.equals("pause")){
            // AttendanceCheck시에 음악 일시 정지
            if(mediaPlayer!=null){
                this.mediaPlayer.pause(); // 음악을 일시정지
                pausePosition = mediaPlayer.getCurrentPosition(); // 음악의 일시정지 타이밍을 저장
                this.vibrator.cancel();
            }
        } else if (state.equals("restart")){
            if(!mediaPlayer.isPlaying()){
                mediaPlayer.seekTo(pausePosition); // 음악이 일시정지된 타이밍을 찾음
                mediaPlayer.start();
                this.vibrator.vibrate(new long[]{500, 1000, 500, 1000}, 0);
            }
        }
        return START_NOT_STICKY;
    }

    @RequiresApi(Build.VERSION_CODES.O)
    private String createNotificationChannel() {
        String channelId = "Alarm";
        String channelName = getString(R.string.app_name);

        NotificationChannel channel = new NotificationChannel(channelId, channelName, NotificationManager.IMPORTANCE_NONE);
        NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        channel.setSound(null, null);
        channel.setLockscreenVisibility(Notification.VISIBILITY_PRIVATE);
        manager.createNotificationChannel(channel);

        return channelId;
    }
}
~~~
> 참고자료 : [`MediaPlayer()`]([https://developer.android.com/guide/topics/media/mediaplayer?hl=ko](https://developer.android.com/guide/topics/media/mediaplayer?hl=ko)), [`Vibrator()`]([https://developer88.tistory.com/103](https://developer88.tistory.com/103)), [`NotificationChannel`]([https://developer.android.com/training/notify-user/channels?hl=ko](https://developer.android.com/training/notify-user/channels?hl=ko))

[AlarmService.java 전체 코드]([https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AlarmService.java](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AlarmService.java)) 

## Diary


 다이어리 기능은 자신의 하루 일과를 적으면서 마무리 할 수 있게 도와주는 기능이다.
 목록보기 칸에서는 자신의 작성한 일기의 수정 및 삭제를 할 수 있다.
 연필 모양 버튼을 클릭 시, 다이어리를 작성할 수 있는 화면이 나오고 제목과 내용을 작성하면 저장버튼이 실행될 수 있게 변한다. 내용을 입력하고 저장버튼을 누르면 눈 모양의 버튼을 클릭하면 자신이 적었던 목록들을 확인할 수 있다.
  - 다이어리 작성칸에서 글을 작성하고 저장하면 목록보기 창에서 본인이 적었던 날짜와 시간과 함께 확인할 수 있다.
  -  아래 사진은 다이어리 실행 화면이다.
 <br>
 ![image](https://user-images.githubusercontent.com/62636101/85548097-8fda1480-b659-11ea-8012-71e8e66f16af.png)
 <br>
 
 다이어리 디자인 구성을 위해 아래와 같이 build.gradle (:app)에 추가해준다.
~~~java
implementation 'com.android.support:design:29.0.0'
~~~

<br>

### 다이어리 작성 , 저장, 목록보기

#### DiaryActivity.java
 다이어리에 글을 작성하고 저장버튼을 누르면 파이어베이스에 저장되면서, 내용을 수정 및 삭제할 수 있는 Activity이다. 
작성 칸에는 제목을 적을 수 있는 칸, 내용을 적을 수 있는 칸 그리고 저장 버튼이 있다.


#### save_btn 설정
- 저장버튼에는 파이어베이스에 같은 시간 동시 저장을 막기 위한 save_btn 1초의 딜레이가 걸려져 있다.
- edit_title과 edit_content가 비었을시 저장버튼이 실행되지 않는다.
 ~~~java
 //save 버튼 클릭 시 1초 동안 딜레이 발생
    save_btn.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            save_btn.setEnabled(false);
            // btn disabled 1sec
            new Handler().postDelayed(new Runnable() {
                public void run() {
                    save_btn.setEnabled(true);
                }
            }, 1000);

            //edit_title과 edit_contents가 비었을시 DB에 삽입 불가
            if(edit_title.getText().toString().getBytes().length<=0 || edit_contents.getText().toString().getBytes().length<=0)
            {

            }else {
                InsertDB();
            }
        }
    }
~~~


#### 작성한 일기내용 파이어베이스에 저장

- 데이터베이스에 저장을 위한 InsertDB 메소드이다.
- 파이어베이스에 edit_title과 edit_contents, writeTime을 저장한다.
~~~java
-  public static void InsertDB() {
    Calendar cal = Calendar.getInstance();
    Date date = cal.getTime();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd-HH:mm:ss");
    String writeTime = sdf.format(date);
    // Firebase에 edit_title, edit_contents, writeTime 저장
    mDatabaseReference = mFirebaseDatabase.getReference().child("diary").child(user.getUid()).child(writeTime);
    mDatabaseReference.setValue(edit_title.getText()+ " / " +  edit_contents.getText());
    edit_title.setText("");
    edit_contents.setText("");
        }
~~~

<br>


#### 목록보기 칸에는 작성한 다이어리를 볼 수 있다. 저장된 다이어리를 얼만큼 클릭하는지에 따라서 다른 이벤트가 발생한다.
#### 버튼을 짧게 클릭 시
- setOnItemClickLinstener를 사용해 다이어리 리스트 중 한개를 클릭시 Intent를 이용해 Diary_update로 화면을 전환된다.
~~~java
list_diary.setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        Intent intent = new Intent(getContext(), Diary_Update.class);
        intent.putExtra("date", data.get(position).getDate());
        intent.putExtra("title", data.get(position).getTitle());
        intent.putExtra("contents", data.get(position).getContents());
        startActivity(intent);
    }
}
~~~

#### 버튼을 길게 누를 시
- setOnItemLongClickListener를 사용해 다이어리 리스트 중 한개를 ***길게*** 클릭 시 AlertDialog를 띄운다.
~~~java
list_diary.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
    @Override
    public boolean onItemLongClick(AdapterView<?> parent, View view, final int position, long id) {
        AlertDialog.Builder alertDialog = new AlertDialog.Builder(getContext());
        alertDialog.setMessage(data.get(position).getTitle() + "을(를) 삭제하시겠습니까?");
        alertDialog.setPositiveButton("삭제", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                String code = data.get(position).getDate();
                deleteDB(code);
                showDB();
            }
        });
        alertDialog.setNegativeButton("취소", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.cancel();
            }
        });
        alertDialog.show();
        return true;
    }
}
~~~
<Br>
<Br>

#### 다이어리 목록에는 작성한 순번, 제목, 내용, 작성시간 추가가 필요하다 

#### 일기  목록 보기 메소드
-  data에 본인의 일기(순번, 제목, 내용, 작성날짜) 추가 
~~~java
public static ArrayList<Diary> showDB() {
    mDatabaseReference = mFirebaseDatabase.getReference().child("diary").child(user.getUid());
    if (mDatabaseReference != null) {
        mFirebaseDatabase.getReference().child("diary").child(user.getUid()).addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                data.clear();
                list_diary.setAdapter(listAdapter);
                int code = 1;
                for (DataSnapshot snapshot : dataSnapshot.getChildren()) {
                    String[] fbData = splitData(snapshot.getValue().toString());
                    Diary diary = new Diary();
                    diary.setCode(code);
                    diary.setTitle(fbData[0]);
                    diary.setContents(fbData[1]);
                    diary.setDate(snapshot.getKey());
                    data.add(diary);
                    code += 1;
                }
                list_diary.setAdapter(listAdapter);
            }

            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {
            }
        });
        return data;
    } else {
        return null;
    }
}
~~~

#### 목록창에서 다이어리를 길게 클릭 시 삭제할 수 있는 기능이 있다.
 - deleteDB 메소드이다.
~~~java
public static void deleteDB(String date) {
    mFirebaseDatabase.getReference().child("diary").child(user.getUid()).child(date).setValue(null);
}
~~~
<br>

>[DiaryActivity.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/DiaryActivity.java)

### 목록보기에서의 다이어리 수정 
***Diary_Update.java***
본인이 작성했던 일기들을 확인할 수 있는 목록창이 있다. 작성했던 리스트 중 한개를 클릭하면 내용을 수정할 수 있는 창이 나온다. 여기에서는 저장, 돌아가기 두 가지의 버튼이 있다.

- findViewById 선언
~~~java
editText1 = (EditText)findViewById(R.id.edit_title_update);
editText2 = (EditText)findViewById(R.id.edit_contents_update);
btn1 = (Button)findViewById(R.id.save_btn_update);
btn2 = (Button)findViewById(R.id.return_btn_update);
~~~
- 아래는 btn1과 btn2의 onClickListener이다.
~~~java
btn1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        updateDB(date);
        finish();
    }
});
btn2.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        finish();
    }
});
~~~

>[Diary_Update.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/Diary_Update.java)

<br>

### 다이어리 페이지 변환
***Tab_pager_Adapter.java***

작성창과 목록보기창을 페이지변환을 통해서 볼 수 있다.
~~~java
public Fragment getItem(int position) {
   switch (position) {
       case 0:
           DiaryActivity.Diary_Write diaryWrite = new DiaryActivity.Diary_Write();
           return diaryWrite;
       case 1:
           DiaryActivity.Diary_List diaryList = new DiaryActivity.Diary_List();
           return diaryList;
       default:
           return null;
   }
}
~~~
>[Tab_pager_Adapter.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/Tab_pager_Adapter.java)

<br>


## Attendance Check

출석체크 기능은 공부를 하기 위해 책상에 앉을 수 있도록 도와주는 기능이다. <br>
Time Table에서 시간표를 설정한 후 지정한 시간에 알람이 울리면 출석체크를 실행한다.<br>
출석체크 방법에는 세 가지가 있다.
- firebase ML Kit를 이용한 사물(책상)인식
- firebase ML Kit를 이용한 Text 인식 
- OpenCV를 이용한 Color Histogram Image Matching

![출석체크](https://user-images.githubusercontent.com/50138845/85498439-a7de7380-b61a-11ea-88e8-e990d0cf33b4.jpg)

우선, firebase ML Kit와 openCV를 사용하기 위해 (2번)내용을 수행해야 한다.


### 출석체크 방법 선택 

#### AttendanceCheckActivity.java

알람이 울릴 때, 어떤 방식으로 출석체크를 할지 선택하면서, 선택한 방식으로 출석체크 후 출석과 결석을 판단해주는 Activity이다.<br>
총 네 가지의 button이 존재한다.

- button을 클릭하면 해당 Activity로 이동하여 출석체크 하는 동안 알람이 일시정지된다. 

아래는 각 button들의 `onClickListener()`이다.
~~~java
btnCheck = (Button)findViewById(R.id.btnCheck);
btnCheck.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        mediaPause();
        startActivityForResult(new Intent(getApplicationContext(), ImageLabelActivity.class), LABEL_ACTIVITY);
    }
});

btnTextCheck = (Button)findViewById(R.id.btnTextCheck);
btnTextCheck.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        mediaPause();
        textRecognition();
    }
});

btnSkip = (Button)findViewById(R.id.btnSkip);
btnSkip.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        mediaPause();
        dialogSkip();
    }
});

btnOpencv = (Button)findViewById(R.id.btnOpencv);
btnOpencv.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        mediaPause();
        startActivityForResult(new Intent(getApplicationContext(), ImageMatchingActivity.class), IMAGE_MATCHING_ACTIVITY);
    }
});
}
~~~

랜덤으로 하나의 영어단어를 함께 넘겨주기위해 `String.xml` 에 영어단어 10가지를 추가하여 배열에 넣어주었다.
~~~java
randomText = getResources().getStringArray(R.array.random_text);  
rnd = new Random();
~~~

>String.xml
>~~~java
><string-array name="random_text">  
> <item>passion</item>  
> <item>wish</item>  
> <item>aspiration</item>  
> <item>peace</item>  
> <item>blossom</item>  
> <item>sunshine</item>  
> <item>cherish</item>  
> <item>smile</item>  
> <item>family</item>  
> <item>rainbow</item>  
></string-array>
>~~~

Text 인식을 이용한 출석체크 버튼을 눌렀을 경우 실행되는 `textRecognition()` method 이다.<br>
랜덤으로 randomText배열에 들어가 있는 영어단어 하나를 함께 **TextRecognitionActivity**로 보내준다. 
~~~java
public void textRecognition(){  
   Intent intent = new Intent(this, TextRecognitionActivity.class );  
   int num = rnd.nextInt(9);  
   intent.putExtra("English", randomText[num]);  
  
   startActivityForResult(intent, TEXT_ACTIVITY);  
}
~~~

Skip button을 제외한 각 버튼들을 클릭하면, `startActivityForResult()`를 통해 Activity마다 다른 requestCode와 함께 해당 Activity로 넘겨준다. 아래는 각 Activity의 requestCode를 정의해준 것이다.
~~~java
// 출석체크시 Activity 구분을 위한 requestCode  
final int LABEL_ACTIVITY = 1;  //사물 인식 Activity
final int TEXT_ACTIVITY = 2;  //Text 인식 Activity
final int IMAGE_MATCHING_ACTIVITY = 3; // Image Matching Activity
~~~


`startActivityForResult()`를 사용하여 다른 Activity를 실행해줬을 경우, `onActivityResult()`를 통해 Activity의 결과를 가져와 출석체크 출결여부를 결정한다.<br>
Activity의 구분은 위에서 함께 넘겨준 requestCode로 구분할 수 있다.

- 출석체크 완료 시, 출석 count를 증가시키고 알람과 Activity를 꺼준다.
- 출석체크 실패 시, 출석체크를 다시 수행할 수 있게 일시정지 되었던 알람이 다시 울린다. 
~~~java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    switch (requestCode) {
        case LABEL_ACTIVITY:

        String label = data.getStringExtra("labeling");

        if(label.equals("Desk") || label.equals("Table") ){
            Toast.makeText(this, "Label 출석체크 완료 : "+label, Toast.LENGTH_SHORT).show();
            alarmOff();
            checkDaysTotal(weeks);
            Log.d(TAG, "실행");
            finish();
        }
        else if(label.equals("BackPressed")){
            Toast.makeText(this, "Label 출석체크 취소", Toast.LENGTH_SHORT).show();
            mediaRestart();
        }
        else {
            Toast.makeText(this, "출석체크 실패 : "+label, Toast.LENGTH_SHORT).show();
            count++;
            mediaRestart();
            Log.d("count_number", ""+count);

            if(count>=3){ // count 가 3일 때 (사물인식 출석체크 3번 실패 시) count = 0으로 셋팅후 textRecognition 메소드 실행(text 인식 출석체크 Activity 실행)
                count = 0;
                Log.d("count_reset", ""+count);
                textRecognition();
            }
        }
        break;
    case TEXT_ACTIVITY :
        boolean checkValue = data.getBooleanExtra("checkValue", false);
        if(checkValue == true){
            Toast.makeText(this, "Text 출석체크 완료", Toast.LENGTH_SHORT).show();
            alarmOff();
            checkDaysTotal(weeks);
            finish();
        } else {
            Toast.makeText(this, "Text 출석체크 취소", Toast.LENGTH_SHORT).show();
            mediaRestart();
        }
        break;
    case IMAGE_MATCHING_ACTIVITY :
        boolean checkMatching = data.getBooleanExtra("checkMatching", false);
        if(checkMatching == true){
            Toast.makeText(this, "ImageMatching 출석체크 완료", Toast.LENGTH_SHORT).show();
            alarmOff();
            checkDaysTotal(weeks);
            finish();
        }else {
            Toast.makeText(this, "ImageMatching 출석체크 취소", Toast.LENGTH_SHORT).show();
            mediaRestart();
        }
    }
}
~~~
Skip button을 클릭시 수행되는 `dialogSkip()` method이다.<br>
AlertDialog를 띄워 출석 여부를 고를 수 있다.
~~~java
public void dialogSkip(){
    activity = this;
    AlertDialog.Builder alertdialog = new AlertDialog.Builder(activity);
    alertdialog.setMessage("출석여부를 고르세요.");

    // 확인버튼 - 결석
    alertdialog.setPositiveButton("결석", new DialogInterface.OnClickListener(){
        @Override
        public void onClick(DialogInterface dialog, int which) {
       Toast.makeText(activity, "결석처리 되었습니다.", Toast.LENGTH_SHORT).show();
       alarmOff();
       finish();
        }
    });
    // 취소버튼
    alertdialog.setNegativeButton("출석", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
       Toast.makeText(activity, "출석처리 되었습니다.", Toast.LENGTH_SHORT).show();
       alarmOff();
       checkDaysTotal(weeks);
       finish();
        }
    });
    alertdialog.setNeutralButton("취소", new DialogInterface.OnClickListener(){
        @Override
        public void onClick(DialogInterface dialog, int id)
        {
       Toast.makeText(activity, "'취소'버튼을 누르셨습니다.", Toast.LENGTH_SHORT).show();
       mediaRestart();
        }
    });
    AlertDialog alert = alertdialog.create();
    alert.setTitle("Skip");
    alert.show();
}
~~~
`alarmOff()`는 alarm을 삭제하는 method이다.
intent의 'state'값에 'off'를 담아 sendBroadcast하며 alarmManger에 pendingintent를 보내어 RequestCode에 일치하는 Alarm을 삭제한다.
~~~java
public void alarmOff(){
    // AlarmReceiver
    Intent intent = new Intent(getApplicationContext(), AlarmReceiver.class);
    intent.putExtra("state","off");
    PendingIntent pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), reqCode, intent, PendingIntent.FLAG_CANCEL_CURRENT);
    alarmManager.cancel(pendingIntent);
    sendBroadcast(intent);
    Log.d("ReqTest", reqCode + " 의 pendingintent 알람이 해제되었습니다.");
}
~~~
`checkDaysTotal()`은 사용자가 출석체크를 하였을 때 Database에 출석체크 값을 변경해주는 method이다.
~~~java
public void checkDaysTotal(int day){...}
~~~
`mediaPause()`와 `mediaRestart()`는 출석체크에서 하나의 출석체크 방법을 클릭하였을 때 잠시 Service에게 Media기능과 Vibrator기능을 일시정지 시키는 기능을 수행한다.
현재 실행되고있는 Service에게 'state'값을 intent에 담아 전송하여 관리한다.
~~~java
sintent = new Intent(context, AlarmService.class);

// media를 pause하기위한 Service호출
public void mediaPause(){
    sintent.putExtra("state", "pause");
    // Oreo(26) 버전 이후부터는 Background 에서 실행을 금지하기 때문에 Foreground 에서 실행해야 함
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        context.startForegroundService(sintent);
    } else {
        context.startService(sintent);
    }
}
~~~
~~~java
// pause된 media를 restart하기 위한 Service호출
public void mediaRestart(){
    sintent.putExtra("state", "restart");
    ...
}
~~~
> Android Oreo 버전 이상부터는 background실행을 하지못하기 때문에 별도로 ForegroundService로 주고받는다.
>[AttendanceCheckActivity.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AttendanceCheckActivity.java)

<br>

## Firebase ML Kit를 이용한 출석체크 

Firebase ML Kit를 이용한 출석체크 기능으로는 **사물인식 (Image Labeling)**, **Text 인식 (Text Recognition)** 이 있다. 
- Firebase ML Kit를 사용하기 위해서는 아래와 같이 `build.gradle`에 정의해주어야 한다.

**build.gradle (:app)**
~~~java
implementation 'com.google.firebase:firebase-core:17.4.2'  
implementation 'com.google.firebase:firebase-ml-vision:24.0.0'  
~~~



**InternetCheck.java**<br>
인터넷 여부를 체크하기 위해 인터넷을 체크하는 Activity를 추가한다.
~~~java
public class InternetCheck extends AsyncTask<Void,Void,Boolean> {

    Consumer consumer;

    public InternetCheck(Consumer consumer){
        this.consumer = consumer;
        execute();
    }

    @Override
    protected Boolean doInBackground(Void... voids) {
        try{
            Socket socket = new Socket();
            socket.connect(new InetSocketAddress("google.com",80),1500);
            socket.close();
            return true;
        }catch (Exception e){
            return false;
        }


    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        super.onPostExecute(aBoolean);
        consumer.accept(aBoolean);
    }

    public interface Consumer {
        void accept(boolean internet);
    }
}
~~~
***
### 사물 인식 (Image Labeling)
- 카메라로 책상을 촬영하여, 책상이 인식되면 출석체크가 완료된다.
- 사물인식 (Image Labeling) 기능을 사용할 때, 보다 안정적이고 빠르게 촬영 후 Detect하기 위해 CameraKit를 사용하였다.
- 촬영한 사진의 Label 값을 AttendanceCheckActivity로 전달한다.

>사물 인식을 통한 출석체크가 수행되는 과정
>1. **AttendanceCheckActivity**에서 사물 인식 기능 button을 클릭하여 **ImageLabelActivity**로 이동한다.
>2. **ImageLabelActivity**에서 cameraView를 통해 책상을 촬영한다.
>3. 촬영된 image에서 설정한 ConfidenceThreshold 값 이상인 Label중 가장 높은 Confidence를 가진 Label을 반환한다.
>4. **ImageLabelActivity**가 종료되면서 반환된 Label을 **AttendanceCheckActivity**로 넘겨준다.
>5. **AttendanceCheckActivity**에서 Label 값이 책상이 맞는지 확인한다. 
 
 Firebase ML Kit의 Image Labeling를 사용하기 위해 아래와 같이 `build.gradle`에 정의해주어야 한다.

**build.gradle (:app)**
~~~java
implementation 'com.google.firebase:firebase-ml-vision-image-label-model:19.0.0' 
~~~
>[Firebase MK Kit Image Labeling 설명 바로가기](https://firebase.google.com/docs/ml-kit/android/label-images)

CameraKit를 사용하기 위해 아래와 같이 `build.gradle`에  정의해주어야 한다.

**build.gradle (:app)**
~~~java
implementation 'com.wonderkiln:camerakit:0.13.1'
~~~
>[CameraKit github 바로가기](https://github.com/CameraKit/camerakit-android)


**ImageLabelActivity.java**
- ImageLabelActivity.java는 cameraKit의 cameraView와 Detect를 수행하는 Button을 사용한다.

>activity_image_label.xml
cameraKit의 cameraView를 사용하기 위하여 아래와 같이 layout에 추가한다.
>~~~java
><com.wonderkiln.camerakit.CameraView  
>  android:id="@+id/camera_view"  
>  android:layout_width="match_parent"  
>  android:layout_height="match_parent"  
>  android:layout_above="@+id/btn_detect">
></com.wonderkiln.camerakit.CameraView>
>~~~
아래 코드는 Detect button의 `onClickListener`이다. button을 클릭하면 cameraView가 실행되고 촬영된다.
~~~java
btnDetect.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v) {
   cameraView.start();
   cameraView.captureImage();
    }
});
~~~

`CameraKitListener()` 부분이다. cameraView에 바로 camera를 띄워 촬영한다.
~~~java
cameraView.addCameraKitListener(new CameraKitEventListener() {
    @Override
    public void onEvent(CameraKitEvent cameraKitEvent) { }

    @Override
    public void onError(CameraKitError cameraKitError) { }

    @Override
    public void onImage(CameraKitImage cameraKitImage) {
   waitingDialog.show();
   Bitmap bitmap = cameraKitImage.getBitmap();
   bitmap = Bitmap.createScaledBitmap(bitmap,cameraView.getWidth(),cameraView.getHeight(), false);
   cameraView.stop();

   runDetector(bitmap);
    }

    @Override
    public void onVideo(CameraKitVideo cameraKitVideo) { }
});
~~~
위의 `CameraKitListener()`에서 사용한 `runDetector()` method이다.<br>
아까 만들어준 InternetCheck.java 를 통해 인터넷을 체크한 후 , image에서 사물 인식 confidenceThreshold를 설정해준다.<br>
설정한 confidenceThreshold의 값보다 높은 값을 가지는 Label이 반환한다.
~~~java
private void runDetector(Bitmap bitmap) {
    final FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);

    new InternetCheck(new InternetCheck.Consumer() {
        @Override
        public void accept(boolean internet) {
       if(internet)
       {
      //인터넷이 있을 때 클라우드 사용
      FirebaseVisionCloudImageLabelerOptions options =
         new FirebaseVisionCloudImageLabelerOptions.Builder()
            .setConfidenceThreshold(0.7f) // 감지된 Label의 신뢰도 설정. 이 값보다 높은 신뢰도의 label만 반환됨
            .build();
      FirebaseVisionImageLabeler detector =
         FirebaseVision.getInstance().getCloudImageLabeler(options);

      detector.processImage(image)
                .addOnSuccessListener(new OnSuccessListener<List<FirebaseVisionImageLabel>>() {
               @Override
               public void onSuccess(List<FirebaseVisionImageLabel> firebaseVisionCloudLabels) {
                   processDataResultCloud(firebaseVisionCloudLabels);
               }
                })
                .addOnFailureListener(new OnFailureListener() {
               @Override
               public void onFailure(@NonNull Exception e) { Log.d("EDMTERROR", e.getMessage()); }
      });

       }
       else
       {
           Toast.makeText(ImageLabelActivity.this, "인터넷을 체크하고 다시 촬영해주세요.", Toast.LENGTH_LONG).show();
           ...
       }
        }
    });
}

~~~
위의 `runDetector()` 에서 사용한 `processDataResultCloud()` method 이다.<br>
`runDetector()`에서 인식한 Label을 넘겨받아 Label 값이 존재할 경우 AttendanceCheckActivity로 Label 값을 넘겨주면서 ImageLabelActivity를 종료한다.
~~~java
private void processDataResultCloud(List<FirebaseVisionImageLabel> firebaseVisionCloudLabels) {
    if(firebaseVisionCloudLabels.size()!=0){
        for(FirebaseVisionImageLabel label : firebaseVisionCloudLabels)
        {
            String labeling = label.getText();

            Intent intent = new Intent();
            intent.putExtra("labeling", labeling);

            setResult(RESULT_OK, intent);
            finish();
        }
    }
    else{
        Intent intent = new Intent();
        intent.putExtra("labeling", "NULL");

        setResult(RESULT_OK, intent);
        finish();
    }
    ...
}
~~~
>[ImageLabelActivity.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/ImageLabelActivity.java)
***

### Text 인식 (Text Recognition)

- 제시된 영어단어를 노트에 따라 적고 촬영하여 두 개의 단어가 일치하면 출석체크가 완료된다.
- 출석체크 완료 시, true 값을 AttendanceCheckActivity에 전달한다.
- 출석체크 실패 시, 재촬영을 요구하는 text를 띄워준다.

>Text 인식을 통한 출석체크가 수행되는 과정
>1. **AttendanceCheckActivity**에서 하나의 영어단어를 랜덤으로 **TextRecognitionActivity**로 이동하면서 넘겨준다.
>2. **TextRecognitionActivity**에서 제시된 영어단어를 노트에 따라 적고 촬영한다.
>3. 촬영된 image에서 단어를 인식하고, 인식된 단어와 제시된 영어단어를 비교하여 같을 시에 true값을 반환한다.
>4. **TextRecognitionActivity**가 종료되면서 반환된 true 값을 **AttendanceCheckActivity**에 전달한다. 
   ( 제시된 영어단어와 같지 않을 시에는 Activity가 종료되지 않으며, 재촬영을 요구하는 text를 띄워준다.)
>5. **AttendanceCheckActivity**에서 ture 값을 받았을 경우 출석체크가 완료된다.

AttendanceCheckActivity에서 전달받은 영어 단어를 textView에 표시해준다.
~~~java
Intent intent = getIntent();  
data = intent.getStringExtra("English");  
textView.setText("똑같이 작성해주세요 : "+ data + "\n");
~~~
사진 촬영 button의 `onClickListener`이다. 
~~~java
captureImageBtn.setOnClickListener(new View.OnClickListener() {  
    @Override  
    public void onClick(View v) {  
        dispatchTakePictureIntent();  
    }  
});
~~~
사진 촬영 button을 눌렀을 경우 실행되는 method이다. 카메라를 실행시켜준다.
~~~java
private void dispatchTakePictureIntent() {  
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);  
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {  
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);  
    }  
}
~~~
카메라로 촬영한 후에 실행되는 method이다.<br>
image를 Bitmap으로 저장하고 imageView에 촬영된 사진을 보여준 후, `detectTextFromImage()` method를 실행한다.
~~~java
@Override  
protected void onActivityResult(int requestCode, int resultCode, Intent data) {  
    super.onActivityResult(requestCode, resultCode, data);  
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {  
        Bundle extras = data.getExtras();  
        imageBitmap = (Bitmap) extras.get("data");  
        imageView.setImageBitmap(imageBitmap);  
        detectTextFromImage();  
    }  
}
~~~
>Firebase ML Kit에 대한 method 설명은 아래 링크를 참고하면서 보면 도움이 될 것이다. <br>
>[Firebase ML Kit Text Recognition 설명 바로가기](https://firebase.google.com/docs/ml-kit/android/recognize-text)

위의 카메라로 촬영한 후에 실행되는 method에서의 `detectTextFromImage()` method이다.<br>
인터넷이 연결되어 있을 때, 촬영된 image에서 Text를 인식하고 성공했을 시에  [`FirebaseVisionText`](https://firebase.google.com/docs/reference/android/com/google/firebase/ml/vision/text/FirebaseVisionText) 객체가 성공 리스너에 전달된다.<br>
 
`displayTextFromImage()` method에 `FirebaseVisionText` 객체를 파라미터로 전달하여 실행한다.
> `FirebaseVisionText` 객체는 이미지에서 인식된 전체 텍스트 및 0개 이상의 [`TextBlock`](https://firebase.google.com/docs/reference/android/com/google/firebase/ml/vision/text/FirebaseVisionText.TextBlock) 객체를 포함한다.
~~~java
private void detectTextFromImage()
{
    new InternetCheck(new InternetCheck.Consumer() {
        @Override
        public void accept(boolean internet) {
            if(internet){
                FirebaseVisionImage firebaseVisionImage = FirebaseVisionImage.fromBitmap(imageBitmap);
                FirebaseVisionTextRecognizer firebaseVisionTextDetector = FirebaseVision.getInstance().getOnDeviceTextRecognizer();
                firebaseVisionTextDetector.processImage(firebaseVisionImage).addOnSuccessListener(new OnSuccessListener<FirebaseVisionText>() {
                    @Override
                    public void onSuccess(FirebaseVisionText firebaseVisionText) {
                        displayTextFromImage(firebaseVisionText);
                    }
                }).addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                        Toast.makeText(TextRecognitionActivity.this, "Error: "+ e.getMessage(), Toast.LENGTH_SHORT).show();
                    }
                });
            }else {
                Toast.makeText(TextRecognitionActivity.this, "인터넷을 체크하고 다시 촬영해주세요.", Toast.LENGTH_LONG).show();
            }
        }
    });
}
~~~
`TextBlock` 를 List에 넣어주고, List의 size가 0일 때는 image에서 Text가 인식되지 않은 것이기 때문에 textView에 재촬영을 요구하는 글을 표시한다.<br>
Text가 인식된 경우에는 Text와 AttendanceCheckActivity에서 전달받은 단어를 `check()` method의 파라미터로 전달하여 실행한다. 
~~~java
private void displayTextFromImage(FirebaseVisionText firebaseVisionText) {
    List<FirebaseVisionText.TextBlock> blockList = firebaseVisionText.getTextBlocks();
    if(blockList.size() == 0){
        textView2.setText("사진에서 단어가 인식되지 않았습니다. 다시 촬영해주세요.");
    }
    else {
        String text = "";
        for(FirebaseVisionText.TextBlock block : firebaseVisionText.getTextBlocks())
        {
            text = block.getText().toLowerCase();
            check(text, data);
        }
    }
}
~~~
제시된 단어와 촬영하여 인식된 단어가 같은지 확인하는 `check()` method이다.<br>
두 단어가 일치할 시, 현재 Activity가 종료되면서 **AttendanceCheckActivity**로 true값을 전달한다.
~~~java
public void check(String text, String data){  
    if(text.equals(data)){  
        checkValue = true;  
   Intent intent = new Intent();  
   intent.putExtra("checkValue", checkValue);  
   setResult(RESULT_OK, intent);  
   finish();  
    }  
    else {  
        checkValue = false;  
   textView2.setText("인식된 단어는 " + text);  
    }  
}
~~~
>[TextRecognitionActivity.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/TextRecognitionActivity.java)

<br>

## OpenCV를 이용한 출석체크

OpenCV를 이용한 출석체크를 하기위해서는, 미리 등록된 5장의 책상 사진이 존재해야한다.<br>
Profile에서 5장의 책상사진을 업로드하면 이 기능을 사용할 수 있다.
- OpenCV의 Color Histogram을 이용하여 두 장의 Image를 비교하는 기능을 구현하였다. 
- Color Histogram은 조명에 영향을 받을 수 있기 때문에 여러 장의 사진을 등록하여 비교한다.
- 등록한 사진과 같은 책상 사진을 찍었지만 출석체크가 되지 않은 경우에는 빛에 의해 사진의 밝기가 변화가 생겼기 때문일 수있다.
- 이러한 문제를 개선하기 위해, 출석체크에 실패했을 경우 해당 사진을 등록하는 책상 사진으로 추가할 수 있다.

>[OpenCV Histogram Compare 설명](https://docs.opencv.org/master/d8/dc8/tutorial_histogram_comparison.html)

>Color Histogram를 통한 출석체크가 수행되는 과정
>1. Profile (설정)에서 5장의 책상 사진을 미리 등록한다.
>2. 등록해놓은 책상 사진과 최대한 같은 각도로 책상을 촬영한다.
>3. 등록돼있는 사진들과 출석체크를 위해 촬영한 사진과 비교하여 일치율을 설정해놓은 Threshold와 비교하여 한 장이라도 만족할 시, **AttendanceCheckActivity**로  true 값을 전달한다.
>4. 등록한 사진 5장 모두 만족하지 않았을 경우, 촬영한 해당 사진을 등록할 것인지 Dialog를 통해 선택할 수 있다.    

### Color Histogram Image Matching 출석체크

**ImageMatchingActivity.java**

Camera를 실행시켜주는 button `onClickListener`이다.
~~~java
btnCamera = (Button)findViewById(R.id.btnCamera);  
btnCamera.setOnClickListener(new View.OnClickListener() {  
    @Override  
    public void onClick(View v) {    
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);  
    startActivityForResult(intent, 0);  
    }  
});  

~~~
Camera로 촬영한 후에 실행되는 method이다. 일치 여부를 나타내는 boolean변수를 false로, 비교를 성공한 image의 개수를 나타내는 count 값을 0으로 초기화해주고 `imageDownload()` method를 실행한다.
~~~java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == 0 && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        capturebmp = (Bitmap) extras.get("data");
        CheckSuccess = false;
        count = 0;
            
        imageDownload();
    }
}
~~~
등록되어 있는 image를 불러와서 방금 촬영한 image와 비교하는 `matching()` method를 실행한다.<br>
등록되어 있는 image는 bitmap 변수에 저장하고 방금촬영한 image는 `matching()`에 파라미터로 전달한다.<br>
한 장의 사진과 비교할 때마다 count값이 증가하며 총 5 장의 사진을 다 비교하고 true값을 반환하였다면, **ImageMatchingActivity**를 종료하고 true 값을 **AttendanceCheckActivity**로 전달한다.
~~~java
public void imageDownload(){
    storageRef.listAll().addOnSuccessListener(new OnSuccessListener<ListResult>() {
        @Override
        public void onSuccess(ListResult listResult) {
            for (StorageReference item : listResult.getItems()) {
            
                final long ONE_MEGABYTE = 1024 * 1024;
                item.getBytes(ONE_MEGABYTE).addOnSuccessListener(new OnSuccessListener<byte[]>() {
                    @Override
                    public void onSuccess(byte[] bytes) {
                   ...
                        bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);

                        matching(capturebmp);
                        count++;

                        if(count== 5){
                            if(CheckSuccess == true){
                           ...
                                Intent intent = new Intent();
                                intent.putExtra("checkMatching", CheckSuccess);
                                setResult(RESULT_OK, intent);
                                finish();
                            }else {
                                ...
                                dialogUpload();
                            }
                        }

                    }
                }).addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception exception) { }
                });
            }
        }
    })
    .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) { }
    });
}
~~~
>[Firebase Storage File Download 사용법](https://firebase.google.com/docs/storage/android/download-files)
>[Firebase Storage File List 가져오기](https://firebase.google.com/docs/storage/android/list-files)

등록된 사진과 촬영한 사진을 비교하는 method이다. 촬영한 사진을 파라미터로 받아와서 등록되어 있는 사진과 비교한다.<br>
이 mathod를 위의 `imageDownload()` 에서 총 5번 수행하여 5장의 사진 모두 비교한다. <br>
각각의 이미지를 bitmap에서 Mat으로 변환을 해주고, [`Imgproc.cvtColor()`](https://docs.opencv.org/master/d8/d01/group__imgproc__color__conversions.html#ga397ae87e1288a81d2363b61574eb8cab)를 통해 HSV로 변환한다.<br>
[`Imgproc.calcHist()`](https://docs.opencv.org/master/d6/dc7/group__imgproc__hist.html#ga4b2b5fd75503ff9e6844cc4dcdaed35d)를 통해 color Histogram을 계산한 후, [`Core.normalize()`](https://docs.opencv.org/master/dc/d84/group__core__basic.html#ga1b6a396a456c8b6c6e4afd8591560d80)로 정규화해준다.<br>
각각 정규화까지 끝난 image를 [`Imgproc.compareHist()`](https://docs.opencv.org/master/d6/dc7/group__imgproc__hist.html#gaf4190090efa5c47cb367cf97a9a519bd)로 color Histogram을 비교하여 일치율을 `metric_val`변수에 넣어준다.<br>
metric_val값이 0에 가까울수록 일치율이 높은 결과이다.<br>
같은 책상사진을 촬영하였을 때, 다른 책상 혹은 다른 곳을 촬영하였을 때 등 여러 테스트를 거쳐 0.2값보다 작은 경우를 일치하는 것으로 판단하도록 구현하였다.<br>
0.2값보다 작은 image가 하나라도 존재한다면 true 값을 반환하여 출석체크가 가능하다.
~~~java
public void matching( Bitmap bitmap2){

    if(!OpenCVLoader.initDebug()){
        Log.d("start error : ", "OpenCV not loaded");
    } else {
        Log.d("start : ", "OpenCV loaded");
        try {
            Mat hist_1 = new Mat();
            Mat hist_2 = new Mat();

            MatOfFloat ranges = new MatOfFloat(0f, 256f);
            MatOfInt histSize = new MatOfInt(25);
       
       //등록된 사진
            img1 = new Mat();
            Utils.bitmapToMat(bitmap, img1);
            Imgproc.cvtColor(img1, img1, COLOR_BGR2HSV);
            Imgproc.calcHist(Arrays.asList(img1), new MatOfInt(0), new Mat(), hist_1, histSize, ranges);
            Core.normalize(hist_1, hist_1, 0, 1, Core.NORM_MINMAX);
       
       //촬영한 사진
            img2 = new Mat();
            Utils.bitmapToMat(bitmap2, img2);
            Imgproc.cvtColor(img2, img2, COLOR_BGR2HSV);
            Imgproc.calcHist(Arrays.asList(img2), new MatOfInt(0), new Mat(), hist_2, histSize, ranges);
            Core.normalize(hist_2, hist_2, 0, 1, Core.NORM_MINMAX);
       
       //두 사진 비교 후 결과
            metric_val = Imgproc.compareHist(hist_1, hist_2, Imgproc.HISTCMP_BHATTACHARYYA);// 0이 일치
            if(metric_val < 0.2) {
                CheckSuccess = true;
            }
            
        } catch (Exception e) { }
    }
}
~~~
출석체크에 실패했을 시 띄워주는 dialog이다. 
- 등록 버튼을 누를 시, 가장 오래된 사진을 하나 삭제하고 촬영한 해당 사진을 등록한다.
- 취소 버튼을 누를 시, 등록을 하지않고 다시 출석체크를 진행해야 한다.

여기서 사용되는 [`checksize()`](https://github.com/JJinTae/MakeYouStudy/blob/c3e8c9d4b3280c0fae93a51494ed29fe4fca873c/app/src/main/java/com/android/MakeYouStudy/ImageMatchingActivity.java#L285)와 [`imageUpload()`](https://github.com/JJinTae/MakeYouStudy/blob/c3e8c9d4b3280c0fae93a51494ed29fe4fca873c/app/src/main/java/com/android/MakeYouStudy/ImageMatchingActivity.java#L256)는 ProfileActivity에 있는 method와 동일하여 Link로 남겨두었다.
~~~java
public void dialogUpload(){
    activity = this;

    AlertDialog.Builder alertdialog = new AlertDialog.Builder(activity);
    alertdialog.setMessage("해당 사진을 등록하시겠습니까?");

    // 등록 버튼
    alertdialog.setPositiveButton("등록", new DialogInterface.OnClickListener(){
        @Override
        public void onClick(DialogInterface dialog, int which) {
            checksize();
            imageUpload(capturebmp);
        }
    });

    // 취소 버튼
    alertdialog.setNegativeButton("취소", new DialogInterface.OnClickListener() {

        @Override
        public void onClick(DialogInterface dialog, int which) {
            Toast.makeText(activity, "취소 되었습니다.", Toast.LENGTH_SHORT).show();
        }
    });


    AlertDialog alert = alertdialog.create();
    alert.setTitle("출석체크 실패");

    alert.show();

}
~~~
>[ImageMatchingActivity.java 전체 코드](https://github.com/JJinTae/MakeYouStudy/blob/c3e8c9d4b3280c0fae93a51494ed29fe4fca873c/app/src/main/java/com/android/MakeYouStudy/ImageMatchingActivity.java)

<br>

## Attendance Rate
MakeYouStudy에서 출석률을 저장하는 역할을 수행한다.<br>
MakeYouStudy에서는 데이터를 Database에서 불러오기 때문에 데이터를 불러오는 Code를 생략하고 Android의 좋은 OpenSource Chart인 [MPAndroidChart]([https://github.com/PhilJay/MPAndroidChart](https://github.com/PhilJay/MPAndroidChart))위주로 알아보도록 하겠다.
> MakeYouStudy에서는 Barchart와 PieChart를 사용하였다.

![출석률 그래프](https://user-images.githubusercontent.com/62635984/85493109-41088c80-b611-11ea-994e-11ae1c7188c7.png)


build.gradle(Module:**app**) dependency에 다음을 추가한다.
~~~java
dependencies {
    implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'
}
~~~
**AttendanceRateActivity in Layout.xml**
~~~java
<com.github.mikephil.charting.charts.PieChart
android:id="@+id/piechart"
android:layout_width="match_parent"
android:layout_height="250dp">
</com.github.mikephil.charting.charts.PieChart>

<com.github.mikephil.charting.charts.BarChart
android:id="@+id/barchart"
android:layout_width="match_parent"
android:layout_height="350dp">
</com.github.mikephil.charting.charts.BarChart>
~~~
**AttendanceRateActivity.Java**
차트의 객체를 선언한다.
~~~java
public class AttendanceRateActivity extends AppCompatActivity {
    // chart 참조 객체 선언
    PieChart pieChart;
    BarChart barChart;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_attendance_rate);
        
        pieChart = (PieChart)findViewById(R.id.piechart);
        barChart = (BarChart)findViewById(R.id.barchart);
~~~
- 데이터를 담을 list또는 변수를 선언
- Barchart의 Y값을 바꾸기 위한 list 선언
- Color를 지정해주기 위한 list 선언
~~~java
ArrayList<BarEntry> Daycheck = new ArrayList<>();  // Barchart에 담을 BarEntry list
ArrayList<PieEntry> yValues = new ArrayList<>();  // Piechart에 담을 PieEntry list
final String[] weekdays = {"Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"}; // Barchart의 Y값을 바꾸기 위한 list

// Color를 지정해주기 위한 list
final int[] checkColor = {ContextCompat.getColor(this, R.color.Check), ContextCompat.getColor(this, R.color.Total)};
final int[] weekColor = {  
   ContextCompat.getColor(this, R.color.Mon),  
   ContextCompat.getColor(this, R.color.Tue),  
   ContextCompat.getColor(this, R.color.Wed),  
   ContextCompat.getColor(this, R.color.Thu),  
   ContextCompat.getColor(this, R.color.Fri),  
   ContextCompat.getColor(this, R.color.Sat),  
   ContextCompat.getColor(this, R.color.Sun)  
};  
~~~
임시 데이터 값
~~~java
int AllCheck = 3;
int AllTotal = 5;
Daycheck.add(new BarEntry(30f, 0));
Daycheck.add(new BarEntry(40f, 1));
Daycheck.add(new BarEntry(30f, 2));
Daycheck.add(new BarEntry(20f, 3));
Daycheck.add(new BarEntry(15f, 4));
Daycheck.add(new BarEntry(50f, 5));
Daycheck.add(new BarEntry(31f, 6));
~~~

~~~java
// PieChart
pieChart.setUsePercentValues(true);  // pieChart를 Percent로 표시할지 설정
pieChart.getDescription().setEnabled(false);  // 
pieChart.setExtraOffsets(5, 5, 5, 5);  
  
pieChart.setDragDecelerationFrictionCoef(0.5f);  
  
pieChart.setHoleColor(Color.WHITE);  
pieChart.setTransparentCircleRadius(55f);  
  
yValues.add(new PieEntry(AllCheck,"출석"));  
yValues.add(new PieEntry(AllTotal,"미출석"));  
  
// 그래프 제목 지우기  
Description piedescription = new Description();  
piedescription.setEnabled(false);  
pieChart.setDescription(piedescription);  
pieChart.getLegend().setEnabled(false);  
  
pieChart.animateY(1500, Easing.EaseOutBounce); // 애니메이션  
  
PieDataSet pieDataSet = new PieDataSet(yValues, "");  
pieDataSet.setSliceSpace(3f);  
pieDataSet.setSelectionShift(12f);  
pieDataSet.setColors(checkColor);  
  
PieData pieData = new PieData(pieDataSet);  
pieData.setValueTextSize(10f);  
pieData.setValueTextColor(Color.YELLOW);  
  
pieChart.setData(pieData);
~~~
Barchart는 Y축이 Left와 Right가 있기 때문에 잘 고려해서 사용하여야 한다.
~~~java
// BarChart
XAxis xAxis = barChart.getXAxis();  // barChart의 X축
YAxis yLAxis = barChart.getAxisLeft();  // barChart의 Left_Y축
YAxis yRAxis = barChart.getAxisRight();  // barChart의 Right_Y축
  
// Y축 오른쪽 비활성화  
yRAxis.setDrawLabels(false);  
yRAxis.setDrawAxisLine(false);  
yRAxis.setDrawGridLines(false);  
  
// Y축 왼쪽 설정  
yLAxis.setDrawLabels(false);  
yLAxis.setDrawAxisLine(false);  
yLAxis.setAxisMaximum(100f);  // Y축의 최댓값을 정해준다.
yLAxis.setAxisMinimum(0f);  // Y축의 최솟값을 정해준다.
// X축 설정  
xAxis.setPosition(XAxis.XAxisPosition.BOTTOM_INSIDE); // x값 표시 위치  
xAxis.setDrawGridLines(false); // x축 GridLinexAxis.setDrawAxisLine(false);  
xAxis.setTextSize(15f);  
xAxis.setValueFormatter(new IndexAxisValueFormatter(weekdays)); // 그래프 Y축 포맷 변경
  
barChart.getDescription().setEnabled(false); // 그래프 제목 삭제  
barChart.getLegend().setDrawInside(false);  // 범례 삭제
barChart.getLegend().setEnabled(false); // 그래프 범례 삭제  
// 그래프 zoom 애니메이션  
barChart.setPinchZoom(false);  
barChart.setScaleEnabled(false);  
barChart.setDoubleTapToZoomEnabled(false);  

barChart.animateY(1500, Easing.EaseOutBounce); // 그래프 애니메이션  
  
BarDataSet bardataset = new BarDataSet(Daycheck, "");  
BarData barData = new BarData(bardataset);  
bardataset.setColors(weekColor);  
  
barChart.setData(barData);
~~~

[AttendanceRateActivity.Java 코드 보러 가기]([https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AttendanceRateActivity.java](https://github.com/JJinTae/MakeYouStudy/blob/master/app/src/main/java/com/android/MakeYouStudy/AttendanceRateActivity.java))
