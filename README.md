<!-- 목차 -->
# 목차
* 어플 소개
* 차별점
* Splash Activity 
* 데이터 베이스
    * 데이터 베이스 생성
    * 데이터 베이스 연동 
* 로그인
* 페이스북 로그인
* 회원정보 띄우기
* 즐겨찾기 
* mapbox 
    * SDK 설치 및 API 발급
    * 지도
    * 길찾기
    * 네비게이션
    * 테마설정
* 장소 자동완성
* Speak To Text
* Menu Bar 만들기
* 언어 설정
* Unity
    * 사용법
    * 안드로이드 연동
    * mapbox AR
<br>

<!-- 어플 소개 -->
# GUIDE DOG
GUIDE DOG은 **증강현실**을 이용한 **길찾기** 애플리케이션으로
길을 찾지 못하는 우리를 목적지까지 안전하게 안내해 주는 안내견 같은 애플리케이션이다.<br>
평면 지도는 사실적이지 않아 건물 내부와 외부를 구별하기 어렵다. <br>
방향 감각이 없는 사람들은 길을 찾는 데 어려움을 겪는다.<br>
이러한 점을 개선하기 위해 증강현실을 이용하여 직관적인 안내로 손쉽게 목적지를 도착하는 애플리케이션을 만들었다.<br>
2차원 공간상의 다양한 지리정보를 3차원 데이터로 표현하여 현실적으로 전달한다.<br>
<!-- 차별점 -->
# 차별점
* 

<br>

<!-- splash Activity -->
# Splash Activity
Splash 화면은 애플리케이션을 켰을 때 **맨 처음** 보여지는 화면으로 1초동안 보여진 후 LoginActivity로 전환 되면서 Login화면이 보여지고 Splash 화면은 종료 된다.<br>
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
     // setContentView(R.layout.activity_splash);
        try {
            Thread.sleep(1000); //대기 시간 설정
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    //1000 대기 후 로그인 액티비티 실행
        startActivity(new Intent(this, LoginActivity.class));
        finish();
    }
}
```
<br><br>
![splash 화면](http://cfile284.uf.daum.net/image/998402475CFA2DFE355622)<br><br><br>

<!-- 데이터베이스 -->
# 데이터베이스
## 1. 데이터베이스 생성

<br>


## 2. 데이터베이스 연동
데이터베이스의 경우 main thread에서접근하면 오류가 생길수 있기 때문에 AsyncTask를 통해 background에서 실행한다.<br>
params로 값을 가져와 변수에 저장한후 URL Connection을 통해 서버에 접근한다.<br>
bufferedWriter을 통해 POST형식으로 변수를 php파일로 보내 파일을 실행한 후 bufferedReader를 통해 결과값을 읽어온다.<br>
이때 UTF-8로 설정해 주어야 한글을 깨짐현상없이 읽어올 수 있다.
```java
@Override
    protected String doInBackground(String... params) {
        String type = params[0];
        if(type.equals("login")) {
            try {
                index = "login";
                String user_id = params[1];
                String password = params[2];
                URL url = new URL("http", "10.0.2.2", 80, "login.php");
                HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
                httpURLConnection.setRequestMethod("POST");
                httpURLConnection.setDoOutput(true);
                httpURLConnection.setDoInput(true);
                OutputStream outputStream = httpURLConnection.getOutputStream();
                BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(outputStream, "UTF-8"));
                String post_data = URLEncoder.encode("user_id", "UTF-8") + "=" + URLEncoder.encode(user_id, "UTF-8") + "&"
                        + URLEncoder.encode("password", "UTF-8") + "=" + URLEncoder.encode(password, "UTF-8");
                bufferedWriter.write(post_data);
                bufferedWriter.flush();
                bufferedWriter.close();
                outputStream.close();
                InputStream inputStream = httpURLConnection.getInputStream();
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream, "UTF-8"));
                String result = "";
                String line = "";
                while ((line = bufferedReader.readLine()) != null) {
                    result += line;
                }
                bufferedReader.close();
                inputStream.close();0
                httpURLConnection.disconnect();

                return result;
            } catch (MalformedURLException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

<br>

<!-- 회원가입 -->
# 회원가입
![회원가입 화면](http://cfile247.uf.daum.net/image/99F778495CFA2ECB3EFA58)<br><br>
회원가입 버튼을 누르면 type, id, password, password 확인, 이름, Login, 전화번호, 개인정보 동의 여부를 BackgroundWorker로 보내고<br>
동시에 BackgroundWorker가 실행되어 받아온 SignupActivity의 TextView의 값을  register.php 파일로 보낸다.<br><br>
```java
public void OnReg(View view) {
    String str_id = id.getText().toString();
    String str_password = password.getText().toString();
    String str_password2 = password2.getText().toString();
    String str_name = name.getText().toString();
    String str_email = email.getText().toString();
    String str_phone = phone.getText().toString();
    String str_agree;

    if(agree.isChecked()){
        str_agree = "yes";
    }
    else{
        str_agree = "no";
    }

    String type = "register";
    BackgroundWorker backgroundWorker = new BackgroundWorker(this);
    backgroundWorker.execute(type, str_id, str_password, str_password2, str_name, str_email, str_phone, str_agree);

}
```
php파일에서 if else문을 이용하여 결과에따라 다른 값을 출력한다.

1.빈칸이 있다면 **"빈칸을 입력해주세요."** <br>
2. id값이 DB에 이미 있는 값이라면 **"이미 사용중인 아이디 입니다."** <br>
3. password값과 password확이값이 다르다면 **"비밀번호와 비밀번호 확인이 다릅니다."**<br>
4. 정보제공 동의에 체크하지 않았다면 **"정보제공에 동의해주세요."**<br>
5. 이상이 없다면 insert 문을 이용하여 DB에 저장한 후 **"회원가입 되었습니다."**<br>

결과를 읽어와 result에 저장하고 onPostExecute함수로 return result한다.<br><br>
onPostExecute함수가 실행되어 result에 저장된 결과값이 알림창으로 뜨고 결과값이 **회원가입 되었습니다.** 이면 LoginActivity로 전환된다. 
```java
else if(result.equals("회원가입 되었습니다.")){
    alertDialog.setMessage(result);
    alertDialog.show();
    new Handler().postDelayed(new Runnable(){
        @Override
        public void run(){
            Intent i = new Intent(context, LoginActivity.class);
            context.startActivity(i);
        }
    },1000);//약 1초뒤에 run() 내부작업 실행
}
```
<br><br>
![DB에 저장](http://cfile279.uf.daum.net/image/990466415CFD01FB49DED0)<br><br>

<!-- 로그인 -->
# 로그인
<br>
1. 로그인 성공<br>

로그인버튼을 누르면 TextView에 type, 입력된 id, password값을 BackgroundWorker로 보낸다.<br>
동시에 BackgroundWorker가 실행되어 AsyncTask를 통해 받아온 값을 login.php파일로 id,password값을 보내 DB에 일치하는 값이 있는지 확인한다.<br><br>
![login](http://cfile242.uf.daum.net/image/996A653F5CFD0306173C62)<br><br>
일치하는 값이 있으면 **"로그인 되었습니다."** , 일치하는 값이 없으면 **"일치하는 회원정보가 없습니다."** 를 읽어와 result에 저장하고 onPostExecute함수로 return result한다.<br><br> onPostExcute함수가 실행되어 result 값이 **"로그인 되었습니다."** 일 경우 type과 id 값을 다시 BackgroundWorker로 보내고<br>
id를 user_info.php파일로 보내 id값에 해당하는 user의 이름과 email값을 **"username : useremail"** 형식으로 읽어와 result값에 저장하고 onPostExecute함수로 return result한다.
```java
if(result.equals("로그인 되었습니다.")){
    new Handler().postDelayed(new Runnable() {
        @Override
        public void run() {
            String type = "get_userInfo";
            BackgroundWorker backgroundWorker = new BackgroundWorker(context);
            backgroundWorker.execute(type, loginActivity.UseridEt.getText().toString());
        }
    }, 500);
}
```

<br>onPostExcute함수가 실행되어 index값이 get_userInfo일 경우 user_info 변수에 result값을 저장하고 MainActivity로 전환된다.

```java
else if(index.equals("get_userInfo")){
    user_info = result;
    Intent i = new Intent(context, MainActivity.class);
    context.startActivity(i);
}
```
<br>
2. 로그인 실패<br><br>
onPostExcute함수가 실행되었을때 index값이 login 이면서 result값이 "로그인 되었습니다."가 아닐 경우<br>
읽어온 result값인 "일치하는 회원정보가 없습니다" 알림을 LoginActivity화면에 띄운다.
<br><br>

<!-- 로그인 Session -->
# Login Session
<br>
1. Session값 등록<br>

로그인이 성공해서 MainActivity로 전환되면 MainActivity에서 LoginActivity에 입력한 아이디 패스워드를 String 값으로 가져와 SharedPreferences 에 저장한다.<br>
이때 이 부분은 if문을 이용하여 Sharedpreferences의 id와 password에 저장되있는 값이 null값일 경우에만 실행되게 하여 자동로그인이 아닌 최초 로그인일때만 실행된다.

```java
if(loginActivity.loginId == null && loginActivity.loginPwd == null && loginActivity.nonMember == 0){

    SharedPreferences auto = getSharedPreferences("auto", Activity.MODE_PRIVATE);
    SharedPreferences.Editor autoLogin = auto.edit();
    autoLogin.putString("inputId", loginActivity.UseridEt.getText().toString());
    autoLogin.putString("inputPwd", loginActivity.PasswordEt.getText().toString());
    autoLogin.putString("inputName", backgroundWorker.user_info);
    autoLogin.putString("inputTeg", null);
    autoLogin.commit();
}
```
<br>
2. 자동로그인<br>

앱이 실행되면 LoginActivity가 실행되는데 
Session에 저장된 값이 있다면 Toast문으로 **"자동로그인입니다."** 를 띄우고 바로 MainActivity로 전환된다.
```java
if (loginId != null && loginPwd != null ) {

    Toast.makeText(LoginActivity.this, loginId + "님 자동로그인 입니다.", Toast.LENGTH_SHORT).show();
    Intent intent = new Intent(LoginActivity.this, MainActivity.class);
    startActivity(intent);
    finish();
}
```
<br>
3. Session값 삭제<br>

로그아웃 버튼을 누를경우 SharedPreferences에 저장된 값을 초기화하여 선언할때 default값으로 설정했던 null값으로 만들어준다.<br>
그 후에 LoginActivity로 전환된다.
이때 Session에는 저장된 값이 없는 상태이다.
```java
    SharedPreferences auto = getSharedPreferences("auto", Activity.MODE_PRIVATE);
    SharedPreferences.Editor editor = auto.edit();
    editor.clear();
    editor.commit();
```
<br>
4. 비회원 접속<br>

LoginActivity에서 비회원 접속 버튼을 통해 접속할 경우 별도의 회원가입이나 로그인 없이 mainActivity로 넘어갈수 있다.<br>이 경우 Session에 로그인 정보가 저장되지 않기 때문에 자동로그인 기능은 실행되지 않는다.<br>

비회원 접속 버튼 클릭시 default값이 0인 변수 nonMember 의 값은 1이 된다.
```java
btnShowLocation.setOnClickListener(new View.OnClickListener() {
    public void onClick(View arg0) {
        nonMember = 1;
    
    }
});
```
<br>
session 등록은 session에 저장된 id 와 password값이 null이면서 nonMember = 0 일때 실행된다.

```java
if(loginActivity.loginId == null && loginActivity.loginPwd == null && loginActivity.nonMember == 0)
```
<br>

<!--facebook 로그인 연동 -->
# facebook 로그인 연동
facebook login api를 사용해서 로그인 했을 때도 일반 로그인과 마찬가지로 세션에 정보가 저장되어<br> 자동로그인이 가능하고네비게이션바 상단에 로그인한 회원 정보를 띄울수 있으며 로그아웃시 LoginActivity로 돌아간다.<br>

facebook login api를 통해 jsonobject object에 저장된 값을 가져와 각각 변수에 저장한 후<br>
일반 로그인을 할 때와 마찬가지로 BackgroundWorker를 통해 MainActivity로 넘어가 Session에 저장될 수 있게 해준다.
```java
@Override
    public void onCompleted(JSONObject object, GraphResponse response) {
        try {
            first_name = object.getString("first_name");
            last_name = object.getString("last_name");
            email = object.getString("email");
            id = object.getString("id");
            image_url = "https://graph.facebook.com/"+id+"/picture?type=normal";

            String type = "facebookLogin";

            backgroundWorker1.execute(type, id, email, first_name);
            RequestOptions requestOptions = new RequestOptions();
            requestOptions.dontAnimate();

//Glide.with(LoginActivity.this).load(image_url).into(circleImageView);
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```

로그아웃시 Session을 지우는것 뿐만 아니라 facebook logout까지 실행해준다.
```java
    LoginManager.getInstance().logOut();
```

<!--로그인한 회원 정보 띄우기 -->
# 로그인한 회원 정보 띄우기
로그인한 회원의 이름과 이메일을 네비게이션 상단에 띄운다.<br><br>
![navigationbar](http://cfile284.uf.daum.net/image/9974CE445CFCF0802344D5)<br><br>
1. 처음 로그인할때<br>
처음 로그인 할때는 BackgroundWorker에서 MainActivity로 넘어오기 때문에 BackgroundWorker의 변수 user_info에 저장한 값을 가져와 userName : userEmail 형식으로 저장된 문장을 split 함수로  ":" 을 기준으로 잘라 userName과 userEmail로 분리한 후 각각의 값을 TextView에 넣어준다.

2. 자동 로그인일때<br>
자동 로그인일때는 처음 로그인할때 Session형성과정에서 SharedPreferences의 loginName에 저장되어있는 값을 가져와 userName : userEmail 형식으로 저장된 문장을 split 함수로  ":" 을 기준으로 잘라 userName과 userEmail로 분리한 후 각각의 값을 TextView에 넣어준다.

```java
if(loginActivity.loginName == null){
    //처음 로그인할때
    if(backgroundWorker.user_info != null) {
        String str = backgroundWorker.user_info;
        String str_name = str.split(":")[0];  //":"를 기준으로 문자열 자름
        String str_email = str.split(":")[1];
        txtname.setText(str_name);
        txtemail.setText(str_email);
        backgroundWorker.user_info=null;
    }
}else{
    //자동 로그인일때
    if(loginActivity.loginName != null) {
        String str = loginActivity.loginName;
        String str_name = str.split(":")[0];
        String str_email = str.split(":")[1];
        txtname.setText(str_name);
        txtemail.setText(str_email);
    }
}
```

<!-- 즐겨 찾기 -->
# 즐겨찾기
자주 가는 장소를 즐겨찾기 목록에 추가해 검색하지 않고 클릭만으로 도착지를 지정할 수 있다.

MainActivity 네비게이션바의 즐겨찾기를 누르면 BackgroundWorker로 type과 사용자 아이디를 보내 데이터베이스에서 아이디가 일치하는 장소를  **"장소1 : 장소2 : 장소3 : 장소4..."** 형식의 문자열로 읽어와 MarkList에 결과값을 저장하고 BookMarklist로 전환된다. 
```java
case R.id.BOOKMARK:
    type ="callList";
    BackgroundWorker backgroundWorker = new BackgroundWorker(this);
    backgroundWorker.execute(type, user_id);
    break;
```

BookMarkList 에서 markList에 저장된 값을 split함수를 이용하여 ":"를 기준으로 잘라 ArrayList에 저장한다.
```java
String str = backgroundWorker.markList;
    if(str != null){
        String[] array = str.split(":");

        for(int i=1 ; i< Integer.parseInt(array[0])+1; i++){
            list.add(array[i]);
        }
        TreeSet<String> list2 = new TreeSet<String>(list); //TreeSet에 list데이터 삽입
        list3 = new ArrayList<String>(list2); //중복이 제거된 HachSet을 다시 ArrayList에 삽입
    }
```
중복을 제거하고 오름차순으로 정렬된 ArratList를 ListView에 저장한다.
```java
    ArrayAdapter adapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, list3) ;

    ListView listview = (ListView) findViewById(R.id.listview1) ;
    listview.setAdapter(adapter) ;
```
ListView에 있는 아이템을 클릭하면 클릭한 view에 있는 값을 변수 getListViewString에 문자열로 저장한다. 아이템이 선택되었다는 것을 나타내기 위해 변수 ListView에 1을 할당해준다고 MainActivity로 전환된다.
```java
 // get TextView's Text.
    getListViewString = (String) parent.getItemAtPosition(position) ;
    ListView =1;


    Intent intent = new Intent(BookMarkList.this, MainActivity.class);
    startActivity(intent);
```

MainActivity로 왔을 때 ListView가 1이라면 버튼의 Text가 "즐겨찾기 삭제"로 변하고 buttonState = "delete" 가 된다.
```java
a_d = (Button)findViewById(R.id.btn_add_delete);

if(bookMarkList.ListView == 1){
    autocompleteFragment.setText(bookMarkList.getListViewString);
    a_d.setText("즐겨찾기 삭제");
    buttonState = "delete";
    bookMarkList.ListView = 0;
}else{
    a_d.setText("즐겨찾기 등록");
    buttonState = "add";
    bookMarkList.ListView = 0;
}

```
즐겨찾기 등록 버튼을 누르면 BackgroundWoker로 type, id, 장소명 그리고 버튼 상태를 보내고 동시에 BackgroundWorker를 통해 bookmark.php파일을 실행하여
데이터베이스에 아이디와 장소를 저장한다. 빈칸이라면 **"내용을 입력하세요."**, 데이터베이스에 이미 있는 데이터라면 **"이미 등록된 장소입니다."**, 데이터가 정상적으로 저장되었다면 **"등록되었습니다."** 를 읽어와 result 값을 저장해 onPostExecute함수에서 result값을 알림으로 띄워준다.

```java
public void onMark(View view) {
    place_mark = txtView.getText().toString();
    type = "bookmark";

    BackgroundWorker backgroundWorker = new BackgroundWorker(this);
    backgroundWorker.execute(type, user_id, place_mark, buttonState);
}
```








<!-- mapbox API 발급 -->
# Mapbox 
mapbox는 구글맵에 대항하는 지도 서비스이다.<br>
GUIDE DOG의 지도 및 길찾기 서비스도 mapbox SDK를 사용하였다.<br><br>

<!-- SDK 설치 및 API 발급 -->
## Mapbox SDK 설치 및 API 발급
[mapbox SDK](https://docs.mapbox.com/android/maps/overview/#install-the-maps-sdk)를 사용하는 방법은 다음과 같다. <br><br>
1. [mabox Login 페이지](https://account.mapbox.com/)에 접속하여 로그인을 한뒤 액세스 토큰을 복사한다.<br><br>
![기본 공개 토큰](http://cfile275.uf.daum.net/image/99B9234E5CFFED7B03971D)<br><br>
2. 모듈 app의 build.gradle에 있는 **dependencies**에 다음과 같은 최신 종속성행을 추가 해 준다.
```
  implementation 'com.mapbox.mapboxsdk:mapbox-android-sdk:7.3.2'
```
3. strings.xml파일에 액세스 토큰을 붙여넣는다.
```xml
<string name="mapbox_access_token">MAPBOX_ACCESS_TOKEN</string>
```
4. 액세스 토큰을 Maps SDK에 전달하기 위하여 응용프로그램 내 onCreate()메소드 내에 액세스 토큰을 배치한다.
```java
Mapbox.getInstance(this, getString(R.string.access_token));
```
5. AndroidManifest.xml에 권한을 추가하여 준다.
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
<!-- 지도 -->

<!-- 길찾기 -->

<!-- 내비게이션 -->

<!-- 테마 변경 -->
## 테마 변경
메뉴 바에 **테마 변경**을 클릭하면 총 6가지의 테마가 나와 사용자의 마음대로 변경할 수 있다.<br><br>
![테마 변경](http://cfile269.uf.daum.net/image/99D9F84F5CFA2E012EB2CE)<br><br>


<!-- 장소 자동완성 -->
# 장소 자동완성
목적지를 입력할 때 목적지가 자동완성 되어 더욱 편리하게 장소를 검색할 수 있도록 하기 위하여 Google Place API를 사용하였다.<br>

API를 사용 하는 방법은 다음과 같다.<br><br>

1. [Google Developers Consle](https://console.developers.google.com/apis/dashboard )에 접속하여 새 프로젝트를 만든다.<br><br>
![새 프로젝트](http://cfile290.uf.daum.net/image/999A2C505CFA08CF1ACD0E)<br><br>
2. 새 프로젝트 생성 완료 후 **API 및 서비스 사용설정**을 클릭하여 추가 설정을 해 준다.<br><br>
![추가설정](http://cfile257.uf.daum.net/image/999BA1485CFA08CC1BBFC3)<br><br>
3. Place를 검색하여 **Places API**를 클릭한다.<br><br>
![Places API](http://cfile260.uf.daum.net/image/994B7F485CFA08CC1F4BED)<br><br>
4. **사용설정**을 클릭한다.<br><br>
![사용설정](http://cfile269.uf.daum.net/image/99FECE485CFA08CD153128)<br><br>
5. API가 활성화 되면 다음 화면에서 **사용자 인증 정보**를 클릭하여 인증 설정을 해준다.<br><br>
![사용자 인증 정보](http://cfile278.uf.daum.net/image/99FC8F485CFA08CD248689)<br><br>
6. **사용자 인증 정보 만들기**를 클릭하여 API 키를 발급받는다.<br><br>
![사용자 인증 정보 만들기](http://cfile278.uf.daum.net/image/9975D8485CFA08CD1D8370)<br><br>
7. API 키 생성이 완료 되면 키 사용의 남용을 막기 위해 키 제한을 해준다.<br><br>
![키 생성 완료](http://cfile247.uf.daum.net/image/995C87485CFA08CE11BAAF)<br><br>
8. 애플리케이션 제한사항을 Android앱으로 설정해준 뒤 패키지 이름과 SHA-1인증서 디지털 지문을 입력하여 Android 앱의 사용량을 제한 해준다.<br><br>
![키제한](http://cfile285.uf.daum.net/image/993E0B485CFA08CE20A439)<br><br>
9. SHA-1 인증서 디지털 지문을 찾는 방법은 다음과 같다.<br><br>
    * Android Studio를 열어 API키를 사용할 패키지를 실행한다
    * 맨 우측에 Gradle을 클릭한다.
    * :app > Tasks > android > singingReport를 클릭한다. <br><br>
    ![SHA-1인증서 찾기](http://cfile278.uf.daum.net/image/994B0D365CFA11210BF8AF)<br><br>
    * 하단에 다음과 같이 SHA-1 인증서 지문이 뜬다. <br><br>
    ![SHA-1인증서](http://cfile288.uf.daum.net/image/99BFC3505CFA08CF28F2CE)<br><br>
10. **API 제한사항**에서 **키 제한**을 선택한 후 콤보박스에서 **Places API**를 선택한 후 저장을 클릭한다.<br><br>
![API 제한사항](http://cfile299.uf.daum.net/image/993FF0485CFA08CE209BF8)<br><br>
11. 다음과 같이 API 키가 생성 되고 키 값을 복사하여 프로젝트에 사용한다.<br><br>
![키 발급](http://cfile250.uf.daum.net/image/99E64E475CFA195A0831D9)<br><br>

모듈 app의 build.gradle에 있는 **dependencies**에 google place 라이브러리를 프로젝트에 사용한다고 추가해 준다.
```
implementation 'com.google.android.libraries.places:places:1.0.0'
implementation 'com.android.support:cardview-v7:28.0.0'
```

MainActivity.java클래스에서 Places를 초기화 해 줄 때 발급받은 키 값을 사용한다.
```java
Places.initialize(getApplicationContext(), "발급받은 키값 입력");
// Create a new Places client instance.
PlacesClient placesClient = Places.createClient(this);
```

다음으로는 AutocompleteSupportFragment를 초기화 해준다.<br>
AutocompleteSupportFragment는 사용자에게 검색상자 UI를 제공해주는 기능이다.<br> 
사용자가 입력하면 이 서비스는 장소에대한 예측을 반환하고<br>
사용자가 선택하면 이 서비스를 통해 응답을 반환한다. <br>
```java
// Initialize the AutocompleteSupportFragment.
AutocompleteSupportFragment autocompleteFragment = (AutocompleteSupportFragment)
                getSupportFragmentManager().findFragmentById(R.id.autocomplete_fragment);

// Specify the types of place data to return.
autocompleteFragment.setPlaceFields(Arrays.asList(Place.Field.ID, Place.Field.NAME));
```

장소가 선택 되었을 때 통지를 받는 Listener를 설정한다.<br>
GUIDE DOG의 경우 textview에 값이 전달 되도록 설정하였다. <br>
```java
autocompleteFragment.setOnPlaceSelectedListener(new PlaceSelectionListener() {
    @Override
    public void onPlaceSelected(Place place) {
    // TODO: Get info about the selected place.                
    txtView.setText(place.getName());
    Log.i(TAG, "Place: " + place.getName() + ", " + place.getId());
    }
 });
 ```

완성되면 다음과 같이 '상명'만 입력하여도 '상명대학교 천안캠퍼스'를 검색 할 수 있다.<br>
![장소자동완성](http://cfile295.uf.daum.net/image/990803455CFA24A92F6648)<br><br>


<!-- Speak To Text -->
# Speak To Text

목적지를 검색할 때 키보드를 치지 않고 음성으로 검색할 수 있는 기능을 구현하기 위하여
Google의 **Speak To Text** 기능을 사용하였다.<br>
메인 화면에서 스피커 모양 버튼을 클릭하면 음성을 인식하는 창이 뜬다.<br>
```java
Button sttButton = (Button)findViewById(R.id.btn_stt);
        sttButton.bringToFront();

        sttButton.setOnClickListener(new View.OnClickListener(){
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);//음성 인식 intent생성
                intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);//데이터 설정
                intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, Locale.ENGLISH);//음성 인식 언어 설정
                try{
                    startActivityForResult(intent,200); //오류가 발생할 수 있는 코드
                }catch (ActivityNotFoundException a){
                    Toast.makeText(getApplicationContext(),"Intent problem", Toast.LENGTH_SHORT).show();//에러 시 수행
                }

            }
        });
```
![스피커 버튼](http://cfile257.uf.daum.net/image/995D994A5CFBDE3531FE0A)
![stt](http://cfile270.uf.daum.net/image/999AC54E5CFBF67D03AC19)<br><br>
음성인식 하는 창이 뜨고 '천안역'이라고 말하면 지도에서 자동으로 천안역까지의 경로를 띄워준다.<br>
![천안역](http://cfile248.uf.daum.net/image/99C3F34E5CFE2AA02BC090)
![경로](http://cfile250.uf.daum.net/image/99DBD74D5CFE2AC62A280F)<br><br>
```java
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) { //음성 인식 결과받음
        super.onActivityResult(requestCode, resultCode, data);
        if(requestCode == 200){
            if(resultCode == RESULT_OK && data != null){
                ArrayList<String> result = data.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS);
                //장소를 AutocompleteSupportFragmene로 가져오기
                STT.setText(result.get(0)); //result.get(0)=목적지

                //인식한 동시에 폴리라인 실행
                startButton.setEnabled(true);
                getPointFromGeoCoder(editText.getText().toString());
                Point origin = Point.fromLngLat(Lo,La);
                Point destination = Point.fromLngLat(destinationX, destinationY);
                getRoute_walking(origin,destination);//폴리라인 그리기
                getRoute_navi_walking(origin,destination);
            }
        }
    }
```
<!-- menu bar 만들기 -->
# Menu Bar(Navigation Drawer)
1. layout 폴더에서 오른쪽 마우스를 클릭하여 새 디렉토리를 만든후 이름은 **menu**라 해준다.<br><br>
![새 디렉토리](http://cfile289.uf.daum.net/image/99299B4A5CFE07BE17D485)<br><br>
2. 새로 생성된 menu에서 오른쪽 마우스를 클릭하여 새 리소스파일을 만든 후 이름은 **drawermenu**라 해준다.<br><br>
![drawermenu](http://cfile293.uf.daum.net/image/9960D23C5CFE04B8121ADD)<br><br>
3. drawable폴더에서 오른쪽 마우스를 클릭하여 Vector Assets를 클릭한다.<br><br>
![vector assets](http://cfile275.uf.daum.net/image/991AAA385CFE033316E97B)<br><br>
4. Clip Art를 더블 클릭하여 적절한 아이콘을 선택해 준다.<br><br>
![icon](http://cfile251.uf.daum.net/image/99D06C385CFE033426355E)<br><br>
5. drawermenu.xml에 다음과 같이 아이디, 아이콘 이미지, 타이틀을 입력한다.
```xml
<item
    android:id="@+id/EXIT"
    android:icon="@drawable/ic_power_settings_new_black_24dp"
    android:title="EXIT" />
```
6. MainActivity.java에서 switch case문으로 클릭 리스너를 완성해 준다. 
```java
@Override
public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {
    switch (menuItem.getItemId()) {
         case R.id.EXIT:
                LoginManager.getInstance().logOut();
                moveTaskToBack(true);
                finish();
                android.os.Process.killProcess(android.os.Process.myPid());
                break;
    }
    mDrawerlayout.closeDrawer(GravityCompat.START);
    return true;
}
```
7. 완성 되면 다음과 같이 메뉴가 생긴다.<br><br>
![메뉴1](http://cfile285.uf.daum.net/image/99A6504B5CFE072025E04E)
![메뉴2](http://cfile284.uf.daum.net/image/9974CE445CFCF0802344D5)<br><br>

<!-- 언어 설정 -->
# 언어설정
login 화면 제일 상단에 있는 한국어와 영어 버튼을 클릭하면 애플리케이션의 언어가 각각 한국어와 영어로 변경 된다.<br><br>
![한국어 영어 버튼](http://cfile258.uf.daum.net/image/99206D375CFB5D100CA164)<br><br>
언어를 설정 하는 방법은 다음과 같다<br><br>
1. Android Studio에 Valus파일에서 마우스 오른쪽 버튼을 클릭하여 새 resource file을 생성한다.<br><br>
![새 리소스 파일 생성](http://cfile258.uf.daum.net/image/9949994F5CFB62C01C26D2)<br><br>
2. File name은 **stirngs.xml**로 해준 뒤 Available qualifiers는 **Locale**을 선택 해 준 뒤 >> 버튼을 클릭한다.<br><br>
![Locale](http://cfile296.uf.daum.net/image/99C41F3B5CFB61ED1CA536)<br><br>
3. 영어의 경우 Laungueage는 **en: English**를 선택하고 Specific Region Only는 **US: United States**를 선택한다.<br><br>
![영어](http://cfile293.uf.daum.net/image/999E5B3B5CFB61ED176511)<br><br>
4. 한국어의 경우 2번과정을 거친후 Language는 **ko: Korean**을 선택하고 Specific Region Only는 **KR: South Korea**를 선택한다.<br><br>
![한국어](http://cfile283.uf.daum.net/image/9930DD3B5CFB61ED14DFCE)<br><br>
5. 다음과 같이 파일이 생성되어 있다.<br><br>
![리소스파일](http://cfile261.uf.daum.net/image/99F69A3B5CFB61EE0EC239)<br><br>
6. Login Button을 예로 들 때 영어와 한국어 stirng.xml파일에 각각 다음과 같은 코드를 작성한다.<br>
 ```xml
<string name="Login_button">login</string>
```
```xml
<string name="Login_button">로그인</string>
```
7. 완성되면 다음과 같다.<br><br>
![한국어 모드](http://cfile298.uf.daum.net/image/9992A3445CFFD9090427D5)
![영어 모드](http://cfile253.uf.daum.net/image/999400445CFFD909048C37)<br><br>


<!-- 유니티 -->
# Unity
## Unity 사용법
1. Unity Hub 다운로드 및 설치 https://unity3d.com/kr/get-unity/download<br><br>
2. Unity Hub 실행 - 설치 - 추가 - 버전 선택(다음) - Android Build Support, Documentation, 한국어 체크(다음) - 약관 동의 - 완료
    - 버전 설치하기 전에 충분한 저장 공간이 있는지 확인하고 확보할 것
    <br>
    
    ![UnityHub Version Download](https://blogfiles.pstatic.net/MjAxOTA2MTFfMjg2/MDAxNTYwMjU0MjEwMDky.p_T8Hu0LWiLJaXcXdswxG1OfkkeW-K_Bv9GPsE_jBfgg.Cck4MTCdtI0EmloPA3ODcasoyaRxIiEJSa_pUQEs_RAg.PNG.119taeyoung/versionInstall.png?type=w2)
3. New Project : Unity Hub - 프로젝트 - 새로 생성 - 템플릿 선택(생성)<br>
    Open Project : Unity Hub - 프로젝트 - 추가 - 폴더 선택<br><br>
![Project](https://blogfiles.pstatic.net/MjAxOTA2MTFfODIg/MDAxNTYwMjU0NDE2MjA5.6_oRVC41qt-fMdz_UfyA21giGkaLWOLxOoFW2aCZzYgg.c1Y0Z35a1135SkLzj25rw365Yk8K2LAoLFsiSzHz8oAg.PNG.119taeyoung/newProject.png?type=w2)
#### 유니티 빌드 환경 설정
4. Build Settings에서 Platform을 Android로 선택하고 다운로드 페이지에 가서 안드로이드 모듈을 다운받는다.<br> 모듈의 다운로드가 완료되면 유니티 프로젝트를 닫고 다시 연다.<br><br>
![Downlad Android Module](http://web.stanford.edu/class/cs11si/images/androidsupport.png)
5. Build Settings에서 Android로 Switch Platform한다.<br>
    - Build app bundle (google play) : 사용자 기기에서 게임의 크기를 줄이고 간소화<br>
    - Development Build : 프로파일러 기능이 활성화되며 자동연결 프로파일러와 스크립트 디버깅 옵션 활성화<br>
    
    ![Switch Platform](https://blogfiles.pstatic.net/MjAxOTA2MTFfMTE0/MDAxNTYwMjU0NzM2Mjgz.qBLue-ISnqc7hyi5MPv63dIJaGqTpK6SprTenRAdnKYg.PBHjVkjvQxcBWezvhgvwXAVpemrbhFYRk1MYC12qiJMg.PNG.119taeyoung/switchPlatform.png?type=w2)<br>
6. Edit>Preferences>Exteranl Tools>Android - JDK Installed with Unity 체크, SDK 경로에 <br>
Android Studio>File>Settings>Appearance & Behavior>System Settings>Android SDK - Android SDK Location 복사하여 넣는다.<br><br>
![Android SDK Location](https://blogfiles.pstatic.net/MjAxOTA2MTFfMjQx/MDAxNTYwMjU1ODQyMDA4.SROSYyNV5jbl6wAEKjhPVpQiYuNXm-UeJHhRKpJ0ioAg.O2OkMvz8vsnGt4htn61AGGgud8XnfxfDP2KhQ6VSUccg.PNG.119taeyoung/sdk.png?type=w2)
![sdk](https://user-images.githubusercontent.com/41332126/59277934-03ea8a00-8c9c-11e9-936f-104555555bad.png)
7. Unity>File>Build Settings>Player Settings>Other Settings>Idenfication - Package Name 지정<br><br>
![Package Name](https://blogfiles.pstatic.net/MjAxOTA2MTFfNDIg/MDAxNTYwMjU3MDUyNDgw.JXxAjRz9wUDiJgpsqvS2syeuv_kYow1MlfQrzUCueQYg.i-OzDHlDWzKpL0CuZcoTerqz-IoFab6PJYapfG9dTIAg.PNG.119taeyoung/packageName.png?type=w2)

<br>

#### 비주얼 스튜디오 Visual Studio - 유니티용 C# 스크립트 편집기 C# script editor for Unity
##### 유니티 최초 설치시 When first installing unity
1. Unity Download Assitant > Choose Components - Microsoft Visual Studio Community _checked_<br><br>
![Unity Download Assistant](https://docs.microsoft.com/ko-kr/visualstudio/cross-platform/media/vstu_download-assistant.png?view=vs-2019)

##### 수동 설치 Manual Installation
2. Visual Studio 설치(Community) https://visualstudio.microsoft.com/ko/downloads/<br>
설치돼있다면 Visual Studio Installer 실행>설치됨-수정><br>

1. 워크로드-모바일 및 게임-Unity를 사용한 게임 개발 칸을 체크하고 수정 버튼을 눌러 Unity Engine을 C# 라이브러리에 확장시킨다.<br><br>
![Unity Studio Installer](https://docs.microsoft.com/ko-kr/visualstudio/cross-platform/media/vstu_unity-workload.png?view=vs-2019)
##### 비주얼을 사용하도록 유니티 구성 Set Unity to use Visual Studio
4. Unity>Edit>Preferences<br>Unity 2018.1부터 비주얼 스튜디오는 유니티의 기본 외부 스크립트 편집기여야 한다.<br>From Unity 2018.1, Visual Studio should be the default script editor for Unity.<br><br>
![Unity>Edit>Preferences](https://docs.microsoft.com/ko-kr/visualstudio/cross-platform/media/vstu_unity-preferences.png?view=vs-2019)

1. External Tools - External Script Editor 드롭다운 목록에서 원하는 Visual Studio 버전이 있을 경우 이를 선택하고, 그렇지 않을 경우 찾아보기... 를 선택한다.<br><br>
![External Tools](https://docs.microsoft.com/ko-kr/visualstudio/cross-platform/media/vstu_unity-external-tools.png?view=vs-2019)
1. 찾아보기...>Visual Studio 설치 경로 내부의 Common7/IDE 경로>devenv.exe(열기)<br><br>
![VisualStudio.exe](https://docs.microsoft.com/ko-kr/visualstudio/cross-platform/media/vstu_browse-for-application.png?view=vs-2019)

<br>

#### 유니티 에디터 인터페이스 Learning the interface
* 에디터의 모양은 개인의 취향이나 수행하는 작업의 타입에 따라 프로젝트 또는 개발자에 따라 편한 방식으로 재배치 가능하다.<br>The look of the editor can be different from one project to the next, and one developer to the next, depending on personal preference and what type of work you are doing.<br><br>
![Unity Interface](https://docs.unity3d.com/kr/2018.1/uploads/Main/Editor-Breakdown.jpg)

* 프로젝트 창 The Project Window<br>
프로젝트에서 사용할 수 있는 에셋 라이브러리가 표시되며 프로젝트로 에셋을 임포트하면 이 창에 나타난다.<br>The Project Window displays your library of assets that are available to use in your project.<br>When you import assets into your project, they appear here.<br><br>
![Project Window](https://docs.unity3d.com/kr/2018.1/uploads/Main/ProjectWindowCallout.jpg)

* 씬 뷰 The Scene View<br>
씬을 시각적으로 탐색하고 편집하며 작업 중인 프로젝트 타입에 따라 3D 또는 2D 원근이 표시된다.<br>The Scene View allows you to visually navigate and edit your scene.<br>The scene view can show a 3D or 2D perspective, depending on the type of project you are working on.<br><br>
![Scene View](https://docs.unity3d.com/kr/2018.1/uploads/Main/SceneViewCallout.jpg)

* 계층 구조 창 The Hierarchy Window<br>
씬의 모든 오브젝트는 Hierarchy에 표시되며 씬에 있는 각 항목마다 Hierarchy에 그 항목이 있으므로 두 창은 본질적으로 연결돼있다.<br>Hierarchy는 개체가 서로 어떻게 연결되어 있는지를 보여준다.<br>The Hierarchy Window is a hierarchical text representation of every object in the scene.<br>Each item in the scene has an entry in the hierarchy, so the two windows are inherently linked.<br>The hierarchy reveals the structure of how objects are attached to one another.<br><br>
![Hierarchy Window](https://docs.unity3d.com/kr/2018.1/uploads/Main/HierarchyWindowCallout.jpg)

* 인스펙터 창 The Inspector Window<br>
현재 선택된 오브젝트의 모든 컴포넌트를 편집할 수 있다. 여러 오브젝트 타입마다 서로 다른 여러 컴포넌트 있다.<br>The Inspector Window allows you to view and edit all the properties of the currently selected object.<br>Because different types of objects have different sets of properties, the layout and contents of the inspector window will vary.<br><br>
![Inspector Window](https://docs.unity3d.com/kr/2018.1/uploads/Main/InspectorWindowCallout.jpg)

* 툴바 The Toolbar<br>
왼쪽에는 씬 뷰와 그 안의 오브젝트들을 조작할 수 있는 기본 툴이 있고, 중앙에는 재생, 일시정지,스텝 컨트롤이 있다.<br>오른쪽 버튼을 통해 유니티 클라우드 서비스 및 유니티 계정에 접근할 수 있으며,<br> 이어서 레이어 메뉴와 에디터 창의 대체 레이아웃을 제공하고 커스텀 레이아웃을 저장할 수 있는 에디터 레이아웃 메뉴가 있다.<br>
The Toolbar provides access to the most essential working features. On the left it contains the basic tools for manipulating the scene view and the objects within it.<br>In the centre are the play, pause and step controls. The buttons to the right give you access to your Unity Cloud Services and your Unity Account, followed by a layer visibility menu,<br> and finally the editor layout menu (which provides some alternate layouts for the editor windows, and allows you to save your own custom layouts).<br>
The toolbar is not a window, and is the only part of the Unity interface that you can’t rearrange.<br><br>
![Toolbar](https://docs.unity3d.com/kr/2018.1/uploads/Main/ToolbarCallout.png)

<br>

#### 유니티 게임씬 구성 Set Unity GameScene
1. 오브젝트 추가 : Hierarchy 우클릭>오브젝트 종류 선택(ex.3D Object)>오브젝트 선택(ex.Capsule)<br>
Add GameObject : Right-click on the Hierarchy window > Select Object Type > Select an object<br><br>
![Add Object](https://t1.daumcdn.net/cfile/tistory/26403C335537AA131E)
<br>

1. 트랜스폼 컴포넌트 조정 : 어떤 오브젝트든 만들고나면 꼭 포지션을 필요한 위치로 초기화해서 확인해준다.<br>
position - x, y, z위치값 | rotation - 기울기와 각도 | scale - 각 축별 크기<br>
Control Transform Component : After you create any object, reset the position to the required position and confirm.<br>
position - x, y, z location values | rotation - slope and angle | scale - size for each axis
<br><br>
![Transform Component](https://t1.daumcdn.net/cfile/tistory/254D02445537B1401C)<br>
![Capsule](https://t1.daumcdn.net/cfile/tistory/2746DF445537B14120)

1. C# 스크립트 컴포넌트 추가 :  오브젝트 선택>인스펙터창-Add Component 또는 프로젝트창>스크립트 선택>오브젝트로 드래그<br>
Add C# Script Component : Select an GameObject > Inspector-Add Component OR Project > Select a script > Drag to the GameObject<br><br>
![Add Script Component](https://t1.daumcdn.net/cfile/tistory/270C324A5537B52F17)
![Add Script Component](https://t1.daumcdn.net/cfile/tistory/232E814F5537B5E21C)

1. C# 스크립트 작성 : 스크립트 선택>더블클릭>비주얼 스튜디오에서 수정 및 작성<br>
값을 자주 혹은 직접 수정, 변경해야 하는 경우의 변수는 스크립트에서 변수의 접근제한자를 public으로 선언하여 메인 에디터 창에서 바로 변수의 값에 접근할 수 있게 해준다.<br>
Write a C# script : Select a script > Double Click > Edit and write in Visual Studio<br>
Variables that require frequent or direct modification of a value can be declared public in the script<br>to allow access to the value of the variable directly from the main editor window.
<br><br>
![Write Script](https://msdnshared.blob.core.windows.net/media/2018/02/image330.png)

1. 게임 실행 : 툴바의 재생버튼을 누르면 게임 화면의 예상 실행 결과를 볼 수 있다.<br>프로젝트를 빌드해서(ctrl+shift+B) 문제가 없다면 안드로이드 기기를 컴퓨터와 연결하여 실제적으로 실행시켜(ctrl+B) 볼 수 있다.<br>
Build And Run : Press the play button of the toolbar to see the expected execution results of the game screen.<br>If your project is built(ctrl+shift+B) and there's no problem, you can actually run(ctrl+B) it connecting to an Android device.<br><br>
![Play Button](https://cdn-images-1.medium.com/max/800/0*hIZxk2xfmyTYCL38.)

<br>

## Unity와 Android Studio 연동
1. Unity>File>Build Settings>Player Settings>Other Settings>Identification에서 Minimum API Level을 연동하려는 안드로이드 스튜디오의 API Level과 일치시킨다.<br><br>
![빌드 세팅](http://cfile283.uf.daum.net/image/99E6B93F5CFD46C3024060)<br><br>
2. Build Settings에서 Android 선택 후 Export Project를 체크하고 유니티 프로젝트 안에 새폴더 경로를 생성하여 Export한다.<br>
    - Build app bundle (google play) : 사용자 기기에서 게임의 크기를 줄이고 간소화<br>
    - Development Build : 프로파일러 기능이 활성화되며 자동연결 프로파일러와 스크립트 디버깅 옵션 활성화<br><br>

    ![빌드 시스템](http://cfile244.uf.daum.net/image/99E6953F5CFD46C30243F8)
2. Android Studio>File>Open>유니티프로젝트>Export폴더 경로에서 아까 export한 프로젝트를 연다.<br><br>
3. app>java>패키지명>UnityPlayerActivity와 assets 우클릭 후 Show in Explorer하여 assets와 jniLibs 폴더를 각각 연동하려는 안드로이드 스튜디오 프로젝트의 똑같은 경로에 복사하여 넣어준다.<br><br>
![파일 옮김](http://cfile289.uf.daum.net/image/99E6BD3F5CFD46C402841D)<br><br>
4. 유니티프로젝트>Export폴더>Export된 프로젝트>libs>unity-classes.jar파일을 복사하여 안드로이드 스튜디오에서 좌측 상단에 Android를 Project로 변경하고 프로젝트명>app>libs에 넣어준다.<br><br>
![파일 옮김](http://cfile273.uf.daum.net/image/99E6613F5CFD46C30262CD)
<br><br>
5. AndroidManifest.xml에 UnityPlayerActivity를 추가하여 프로세스를 따로 할당해주고 권한을 설정한다.
    ```java
    <activity android:name=".UnityPlayerActivity" />
    ```
    ```java
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.CAMERA" />
    ```
    <br>
6. build.gradle(Module: app)에 jar 파일형식을 사용하겠다고 명시한다.
    ```java
    allprojects {
        repositories {
            flatDir {
                dirs 'libs'
            }
        }
    }
    ```
    <br>                 
7. File>Sync Project with Gradle Files<br><br>
![Sync Gradle](https://cdn-images-1.medium.com/max/1600/1*UdbZAJNUnmLo4kOPSqqNNQ.png)<br><br>
8. MainActivity에서 Intent를 이용하여 UnityPlayerActivity 클래스를 호출한다.
    ```java
    @Override
    public void onClick(View v) {
        Intent intent = new Intent(getApplicationContext(), UnityPlayerActivity.class);
        startActivity(intent);
        ...
    }
    ```
