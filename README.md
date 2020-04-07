## Writeup Natas

- Natas 0
  ctrl + u ta thấy password natas1: gtVrDuiDfck831PqWsLEZy5gyDz1clto

- Natas 0 -> 1
  vô hiệu hóa javascript trên trình duyệt là có thể chuột phải, hoặc bấm ctrl + u :v 
  pass: ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi

- Natas 1 -> 2
  vẫn ctrl+u thấy có một đường dẫn đến file ảnh /files/pixel.png. Thế nên mình truy cập đến url /files thôi, xem thử có gì không
  thì thấy có thêm một file users.txt. Mở ra thì thấy natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14

- Natas 2 -> 3
  Ctrl + u ra dòng này: No more information leaks!! Not even Google will find it this time...
  Vậy là file robots.txt rồi. Xem thử thì thấy url /s3cr3t/ vào thấy file users.txt ra natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

- Natas 3 -> 4
  Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
  Lúc đầu vào nó hiện ra thông báo này, nó nói là chúng ta truy cập từ "", bạn nên truy cập từ natas5.., nhấn vào link refresh page thì
  Access disallowed. You are visiting from "http://natas4.natas.labs.overthewire.org/" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"

  Xem burp thì thấy dòng Referer: http://natas4.natas.labs.overthewire.org/ vậy giờ thay đổi link như hướng dẫn là được
  kết quả: Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq

- Natas 4 -> 5
  Access disallowed. You are not logged in
  Xem thử cookie thấy có trường loggedin đang có giá trị 0, sửa lại thành 1 
  kết quả: Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1

- Natas 5 -> 6
  ```php
  include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
  ```
  Khi ta nhập đúng secret thì ok. Thế secret ở đâu ? ta thấy dòng - include "includes/secret.inc"; nó lấy từ file này, vậy ta thử truy cập
  url này xem thế nào, ta có: $secret = "FOEIUWGHFEEUHOFUOIU"; giờ nhập vô là ok 
  kết quả: Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9 

- Natas 6 -> 7
  ta thấy trong web có hai button dẫn đến hai đường link, home và about có địng dạng index.php?page= , mà thêm câu gợi ý 
  hint: password for webuser natas8 is in /etc/natas_webpass/natas8 , thế là web này bị lỗi local file inclusion, ta có url
  index.php?page=/etc/natas_webpass/natas8 thế là ra pass DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe

- Natas 7 -> 8
  source code
  ```php
  $encodedSecret = "3d3d516343746d4d6d6c315669563362";

  function encodeSecret($secret) {
      return bin2hex(strrev(base64_encode($secret)));
  }

  if(array_key_exists("submit", $_POST)) {
      if(encodeSecret($_POST['secret']) == $encodedSecret) {
      print "Access granted. The password for natas9 is <censored>";
      } else {
      print "Wrong secret";
      }
  }
  ```
  mình nhập vào input rồi nó xào nặn bằng hàm encodeSecret, sau khi sào nặn xong giống với $encodedSecret
  cho từ đầu thì có password. Vậy giờ ta viết đoạn decode hàm trên, ta có 

  ```php
  $secret = "3d3d516343746d4d6d6c315669563362";
  echo base64_decode(strrev(hex2bin($secret)));
  ```
  ra được kết quả: oubWYf2kBq , nhập vào input ta được Access granted. The password for natas9 is W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl

- Natas 8 -> 9 
  Ta có source
  ```php
  $key = "";

  if(array_key_exists("needle", $_REQUEST)) {
      $key = $_REQUEST["needle"];
  }

  if($key != "") {
      passthru("grep -i $key dictionary.txt");
  }
  ```
  chương trình hoạt động như sau: Mình nhập input thì nó gán vào biến $key và được vào thực thi câu lệnh
  grep -i $key dictionary.txt, điều đáng nói ở đây là không có kiểm tra đầu vào, người nhập muốn nhập gì nhập. Vậy ta sẽ nhập câu lệnh
  khác thử, ta dùng: ; ls /etc ; => ta thấy thư mục natas_webpass, lần mò vào trong ta được câu lệnh cuối cùng
  ; cat /etc/natas_webpass/natas10;  => nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu

- Natas 9 -> 10 
  Dạng bài trước nhưng đã có kiểm tra input , nó trong cho dùng ; nữa thì mình đổi cái khác
  .* cat /etc/natas_webpass/natas11 #
  kết quả: U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK

- Natas 10 -> 11
  ```php
  $defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

  function xor_encrypt($in) {
      $key = '<censored>';
      $text = $in;
      $outText = '';

      // Iterate through each character
      for($i=0;$i<strlen($text);$i++) {
      $outText .= $text[$i] ^ $key[$i % strlen($key)];
      }

      return $outText;
  }

  function loadData($def) {
      global $_COOKIE;
      $mydata = $def;
      if(array_key_exists("data", $_COOKIE)) {
      $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
      if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
          if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
          $mydata['showpassword'] = $tempdata['showpassword'];
          $mydata['bgcolor'] = $tempdata['bgcolor'];
          }
      }
      }
      return $mydata;
  }

  function saveData($d) {
      setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
  }

  $data = loadData($defaultdata);

  if(array_key_exists("bgcolor",$_REQUEST)) {
      if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
          $data['bgcolor'] = $_REQUEST['bgcolor'];
      }
  }

  saveData($data);
  ```

 