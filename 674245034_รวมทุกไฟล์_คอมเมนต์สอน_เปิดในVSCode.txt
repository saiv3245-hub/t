คู่มือรวมโค้ด 674245034 — เปิดไฟล์เดียวใน VS Code
ใช้สำหรับอ่าน/ท่องพรี ไม่ใช่ไฟล์ที่เอาไปรันแทน Project จริง
รูปแบบ Comment: อธิบายว่าอะไรเป็นตัวแปรที่เราสร้างเอง / ศัพท์เฉพาะของภาษา / ใช้ทำอะไร / เชื่อมไฟล์ไหน
ศัพท์ที่ซ้ำกันจะอธิบายสั้นลงในไฟล์ถัด ๆ ไป เพื่อให้จำครบภายใน 1 วัน

==============================================================================================================
ไฟล์ 1/22 : connect/conn.php
หน้าที่: เชื่อม PHP กับ MySQL แล้วสร้าง $conn ให้ไฟล์อื่นใช้ Query ฐานข้อมูล
เชื่อม: ถูก include โดย login/register/index/books/cart/checkout และหน้า Admin เกือบทั้งหมด
FLOW จำ: Host/User/Password/Database → new mysqli → $conn → utf8mb4 → เช็ก Error
==============================================================================================================
<?php

// ตัวแปร 4 ตัวนี้เราสร้างเอง ใช้เก็บข้อมูลสำหรับเชื่อม MySQL
$hostname = "localhost";   // ที่อยู่ MySQL, localhost = อยู่เครื่องเรา
$username = "root";        // Username ของ MySQL
$password = "";            // Password MySQL, "" = ไม่มี Password
$database = "bookstore";   // ชื่อ Database ของระบบเรา

// $conn = ตัวแปรที่เราสร้างเอง ใช้เก็บ Connection หรือการเชื่อม PHP ↔ MySQL
// new และ mysqli = ศัพท์เฉพาะของ PHP/MySQLi; mysqli ใช้สร้างการเชื่อมกับ MySQL
$conn = new mysqli(
    $hostname,
    $username,
    $password,
    $database
);

// mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL
// " " = Double Quote ครอบ String ของ PHP; ' ' = Single Quote ตรงนี้ครอบค่า utf8mb4 ใน SQL
// SET NAMES 'utf8mb4' = ตั้งชุดตัวอักษรให้รองรับภาษาไทย/Unicode
mysqli_query($conn, "SET NAMES 'utf8mb4'");

// if = เงื่อนไขของ PHP; -> = ใช้เข้าไปเรียกข้อมูลใน Object
// connect_error = ข้อมูล Error ของ mysqli ใช้เช็กว่าการเชื่อม MySQL ผิดพลาดหรือไม่
if ($conn->connect_error) {
    // die() = หยุดการทำงานและแสดง Error; . = ใช้ต่อข้อความใน PHP
    // "Connection failed: " = ข้อความที่เราเขียนเอง แปลว่า "การเชื่อมต่อล้มเหลว:"
    die("Connection failed: " . $conn->connect_error);
}

?>

==============================================================================================================
ไฟล์ 2/22 : login.php
หน้าที่: รับ Email/Password ตรวจตาราง users แล้วจำสถานะ Login ใน Session
เชื่อม: include conn.php; Form ส่งกลับ login.php; Admin → book.php; Member → index.php; ลิงก์ register/cart/books/my_orders
FLOW จำ: Session → POST → SELECT users → query → fetch → Session → role → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // $message = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อความแจ้ง Error/สถานะ
    $message = "";

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["login"] = ค่าจาก Form method="post"
    if(isset($_POST["login"])){
        // $email = ตัวแปรที่เราสร้างเอง ใช้เก็บEmail จาก Form
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $email = mysqli_real_escape_string($conn,$_POST["email"]);
        // $password = ตัวแปรที่เราสร้างเอง ใช้เก็บPassword ตามบริบทของไฟล์
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $password = mysqli_real_escape_string($conn,$_POST["password"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_user = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_user = "select * from users where email = '$email' and password = '$password'";
        // $result_user = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_user = mysqli_query($conn,$sql_user);
        // $user = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูล User 1 แถว
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $user = mysqli_fetch_assoc($result_user);

        if($user){
            // "user_id" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["user_id"] = $user["user_id"];
            // "full_name" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["full_name"] = $user["full_name"];
            // "role" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["role"] = $user["role"];

            // == = เปรียบเทียบว่าเท่ากันหรือไม่
            if($user["role"] == "admin"){
                // header("Location: ...") = Redirect Browser ไป book.php
                header("Location: book.php");
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
                // header("Location: ...") = Redirect Browser ไป index.php
                header("Location: index.php");
            }
            // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
            exit();
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            $message = "อีเมลหรือรหัสผ่านไม่ถูกต้อง";
        }
    }
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>เข้าสู่ระบบ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }

        /* .banner = Class ที่เราตั้งเองสำหรับ Banner/พื้นหลัง */
        .banner{
            height: 350px;
            background-image: url("./img/bg.png");
            background-size: cover;
            background-position: center;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <a href="login.php" class="btn btn-outline-dark me-2">
            <i class="fa-solid fa-right-to-bracket"></i>
            เข้าสู่ระบบ
        </a>

        <a href="register.php" class="btn btn-outline-dark me-2">
            <i class="fa-solid fa-user-plus"></i>
            สมัครสมาชิก
        </a>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>
           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>
           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>
           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    <div class="container mt-5 mb-5">
        <div class="row justify-content-center">
            <div class="col-md-5">
                <h2 class="fw-bold text-center mb-4"> เข้าสู่ระบบ </h2>
                <?php
                    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
                    if($message != ""){
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<div class="alert alert-danger">' . $message . '</div>';
                    }
                ?>
                <!-- form ส่งข้อมูลไป login.php แบบ POST -->
                <form action="login.php" method="post">
                    <label class="fw-bold mb-2"> อีเมล </label>
                    <!-- name="email" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["email"] -->
                    <input type="email" name="email" class="form-control mb-3" required>
                    <label class="fw-bold mb-2"> รหัสผ่าน </label>
                    <!-- name="password" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["password"] -->
                    <!-- type="password" = ซ่อนตัวอักษรตอนพิมพ์ -->
                    <input type="password" name="password" class="form-control mb-3" required>
                    <!-- name="login" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["login"] -->
                    <!-- type="submit" = กดแล้วส่ง Form ตาม action/method -->
                    <button type="submit" name="login" class="btn btn-dark w-100">
                        <i class="fa-solid fa-right-to-bracket"></i> เข้าสู่ระบบ </button>
                </form>
                <p class="text-center mt-3">ยังไม่มีบัญชี? <a href="register.php"> สมัครสมาชิก </a> </p>
            </div>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 3/22 : register.php
หน้าที่: รับข้อมูลสมัครสมาชิก ตรวจรหัสผ่านและ Email ซ้ำ แล้ว INSERT users
เชื่อม: include conn.php; Form ส่งกลับ register.php; สมัครสำเร็จ → login.php; Navbar เชื่อม login/logout/cart/index/books/my_orders
FLOW จำ: POST → เช็กรหัสผ่าน → SELECT Email ซ้ำ → num_rows → INSERT users → login.php
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // $message = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อความแจ้ง Error/สถานะ
    $message = "";

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["register"] = ค่าจาก Form method="post"
    if(isset($_POST["register"])){
        // $full_name = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อ-นามสกุล
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $full_name = mysqli_real_escape_string($conn,$_POST["full_name"]);
        // $email = ตัวแปรที่เราสร้างเอง ใช้เก็บEmail จาก Form
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $email = mysqli_real_escape_string($conn,$_POST["email"]);
        // $password = ตัวแปรที่เราสร้างเอง ใช้เก็บPassword ตามบริบทของไฟล์
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $password = mysqli_real_escape_string($conn,$_POST["password"]);
        // $confirm_password = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสผ่านยืนยัน
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $confirm_password = mysqli_real_escape_string($conn,$_POST["confirm_password"]);
        // $phone = ตัวแปรที่เราสร้างเอง ใช้เก็บเบอร์โทร
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $phone = mysqli_real_escape_string($conn,$_POST["phone"]);
        // $address = ตัวแปรที่เราสร้างเอง ใช้เก็บที่อยู่
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $address = mysqli_real_escape_string($conn,$_POST["address"]);

        if($password != $confirm_password){
            $message = "รหัสผ่านไม่ตรงกัน";
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
            // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
            // $sql_check = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
            $sql_check = "select * from users where email = '$email'";
            // $result_check = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
            // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
            $result_check = mysqli_query($conn,$sql_check);

            // mysqli_num_rows() = Function ของ MySQLi ใช้นับว่าผล SELECT มีทั้งหมดกี่แถว
            if(mysqli_num_rows($result_check) > 0){
                $message = "อีเมลนี้ถูกใช้งานแล้ว";
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
                // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
                // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
                // $sql_user = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
                $sql_user = "insert into users
                (full_name,email,password,phone,address,role)
                values
                ('$full_name','$email','$password','$phone','$address','member')";

                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                if(mysqli_query($conn,$sql_user)){
                    // header("Location: ...") = Redirect Browser ไป login.php
                    header("Location: login.php");
                    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
                    exit();
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
                    $message = "ไม่สามารถสมัครสมาชิกได้";
                }
            }
        }
    }
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>สมัครสมาชิก</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>
           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>
           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>
           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    <div class="container mt-5 mb-5">
        <div class="row justify-content-center">
            <div class="col-md-6">
                <h2 class="fw-bold text-center mb-4"> สมัครสมาชิก </h2>
                <?php
                    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
                    if($message != ""){
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<div class="alert alert-danger">' . $message . '</div>';
                    }
                ?>
                <!-- form ส่งข้อมูลไป register.php แบบ POST -->
                <form action="register.php" method="post">
                    <label class="fw-bold mb-2"> ชื่อ - นามสกุล </label>
                    <!-- name="full_name" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["full_name"] -->
                    <input type="text" name="full_name" class="form-control mb-3" required>
                    <label class="fw-bold mb-2"> อีเมล </label>
                    <!-- name="email" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["email"] -->
                    <input type="email" name="email" class="form-control mb-3" required>
                    <label class="fw-bold mb-2"> รหัสผ่าน </label>
                    <!-- name="password" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["password"] -->
                    <!-- type="password" = ซ่อนตัวอักษรตอนพิมพ์ -->
                    <input type="password" name="password" class="form-control mb-3" required>
                    <label class="fw-bold mb-2"> ยืนยันรหัสผ่าน </label>
                    <!-- name="confirm_password" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["confirm_password"] -->
                    <!-- type="password" = ซ่อนตัวอักษรตอนพิมพ์ -->
                    <input type="password" name="confirm_password" class="form-control mb-3" required>
                    <label class="fw-bold mb-2"> เบอร์โทรศัพท์ </label>
                    <!-- name="phone" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["phone"] -->
                    <input type="text" name="phone" class="form-control mb-3">
                    <label class="fw-bold mb-2"> ที่อยู่ </label>
                    <!-- name="address" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["address"] -->
                    <textarea name="address" class="form-control mb-3" rows="3"></textarea>
                    <!-- name="register" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["register"] -->
                    <!-- type="submit" = กดแล้วส่ง Form ตาม action/method -->
                    <button type="submit" name="register" class="btn btn-dark w-100">
                        <i class="fa-solid fa-user-plus"></i> สมัครสมาชิก </button>
                </form>
                <p class="text-center mt-3"> มีบัญชีแล้ว?
                    <a href="login.php">เข้าสู่ระบบ</a>
                </p>
            </div>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 4/22 : logout.php
หน้าที่: ออกจากระบบโดยล้าง Session แล้วกลับหน้าแรก
เชื่อม: รับ Session ที่สร้างจาก login.php → ล้าง → index.php
FLOW จำ: session_start → session_unset → session_destroy → index.php
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();

    // session_unset() = ล้างตัวแปรทั้งหมดใน Session รวม user_id/role/cart
    session_unset();
    // session_destroy() = ทำลาย Session ปัจจุบันบน Server
    session_destroy();

    // header("Location: ...") = Redirect Browser ไป index.php
    header("Location: index.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 5/22 : index.php
หน้าที่: หน้าแรก แสดง Banner ค้นหา หนังสือขายดี และหนังสือมาใหม่
เชื่อม: include conn.php; Search → books.php; หนังสือ → book_detail.php; Navbar → login/register/logout/cart/books/my_orders
FLOW จำ: Session+DB → Query ขายดี/มาใหม่ → แสดงหนังสือ → JS เลื่อนรายการ
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>โลกเหนือหน้ากระดาษ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }

        /* .banner = Class ที่เราตั้งเองสำหรับ Banner/พื้นหลัง */
        .banner{
            height: 350px;
            background-image: url("./img/bg.png");
            background-size: cover;
            background-position: center;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>

           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>

           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>

           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    <section class="banner">
        <div class="text-white" style="text-shadow:2px 2px 5px black;">
            <h1 class="display-4 fw-bold">โลกเหนือหน้ากระดาษ</h1>
            <p class="lead">ค้นพบเรื่องราวในทุกๆหน้าที่พลิกเปิด</p>
            <!-- form ส่งข้อมูลไป books.php แบบ GET -->
            <form action="books.php" method="get">
                <div class="input-group mx-auto" style="max-width: 500px;">
                 <!-- name="search" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_GET["search"] -->
                 <input type="text" class="form-control" name="search" placeholder="ค้นหาชื่อหนังสือ หรือชื่อผู้แต่ง">
                 <button class="btn btn-dark" type="submit">
                    <i class="fa-solid fa-magnifying-glass"></i>
                    ค้นหา
                 </button>
                </div>
            </form>
        </div>
    </section>
    <section class ="container-fluid mt-3 px-3">
        <h2 class="fw-bold mb-4 ms-5">หนังสือขายดี</h2>

        <div class="d-flex align-items-center gap-2">
          <!-- id="bookLeft" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
          <button class="btn btn-outline-dark" type="button" id="bookLeft">
            <i class="fa-solid fa-chevron-left"></i>

        </button>
         <!-- id="bookList" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
         <div class="bookrow d-flex flex-nowrap overflow-auto gap-3 flex-grow-1" id="bookList">
             <?php
                // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
                // SUM()=รวมค่า, GROUP BY=จัดกลุ่ม, AS=ตั้งชื่อผลลัพธ์ชั่วคราว
                // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
                // LIMIT=จำกัดจำนวนแถวที่เอามาแสดง
                // $sql_bestbook = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                $sql_bestbook = "select books.*, sum(order_details.quantity) as total_sale from books inner join order_details on books.book_id = order_details.book_id group by books.book_id order by total_sale desc limit 10";
                // $result_bestbook = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                $result_bestbook = mysqli_query($conn,$sql_bestbook);

                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                while($book = mysqli_fetch_assoc($result_bestbook)){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="border p-2 flex-shrink-0" style="width: 200px;">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<a href="book_detail.php?book_id=' . $book["book_id"] . '">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<img src="./uploads/books/' . $book["cover_image"] . '" width="100%" height="250" style="object-fit: cover;">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '</a>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<h6 class="mt-2 fw-bold">' . $book["book_title"] . '</h6>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<p class="text-muted mb-2">' . $book["author_name"] . '</p>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<p class="fw-bold mb-0">' . $book["price"] . ' บาท</p>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '</div>';
                }
             ?>   
         </div>
           <!-- id="bookRight" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
           <button class="btn btn-outline-dark" type="button" id="bookRight">
             <i class="fa-solid fa-chevron-right"></i>

           </button>
        </div>
    </section>
    <section class="container-fluid mt-5 px-3">
        <h2 class="fw-bold mb-4 ms-5">หนังสือมาใหม่</h2>
        <div class="d-flex align-items-center gap-2">
            <!-- id="newBookLeft" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
            <button class="btn btn-outline-dark" type="button" id="newBookLeft">
                <i class="fa-solid fa-chevron-left"></i>
            </button>
        
        <!-- id="newBookList" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <div class="newbookrow d-flex flex-nowrap overflow-auto gap-3 flex-grow-1" id="newBookList">
            <?php 
                // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
                // LIMIT=จำกัดจำนวนแถวที่เอามาแสดง
                // $sql_newbook = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                $sql_newbook = "select * from books
                       order by book_id desc limit 10";
                // $result_newbook = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                $result_newbook = mysqli_query($conn,$sql_newbook);
                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                while($book = mysqli_fetch_assoc($result_newbook)){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="border p-2 flex-shrink-0" style="width: 200px;">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<a href="book_detail.php?book_id=' . $book["book_id"] . '">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<img src="./uploads/books/' . $book["cover_image"] . '" width="100%" height="250" style="object-fit: cover;">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '</a>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<h6 class="mt-2 fw-bold">' . $book["book_title"] . '</h6>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<p class="text-muted mb-2">' . $book["author_name"] . '</p>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<p class="fw-bold mb-0">' . $book["price"] . ' บาท</p>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '</div>';
                }
            ?>
        </div>
        <!-- id="newBookRight" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-outline-dark" type="button" id="newBookRight">
            <i class="fa-solid fa-chevron-right"></i>
        </button>
        </div>
    </section>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
        
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };

        // getElementById("bookLeft") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("bookLeft").onclick = function(){
            // getElementById("bookList") = หา Element จาก id ที่เราตั้งใน HTML
            // scrollBy() = เลื่อนตำแหน่ง Scroll ของรายการหนังสือ
            document.getElementById("bookList").scrollBy({
                // left บวก=เลื่อนไปขวา, ติดลบ=ซ้าย; smooth = เลื่อนแบบนุ่ม
                left: -300,behavior: "smooth"
            });
        };

        // getElementById("bookRight") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("bookRight").onclick = function(){
            // getElementById("bookList") = หา Element จาก id ที่เราตั้งใน HTML
            // scrollBy() = เลื่อนตำแหน่ง Scroll ของรายการหนังสือ
            document.getElementById("bookList").scrollBy({
                // left บวก=เลื่อนไปขวา, ติดลบ=ซ้าย; smooth = เลื่อนแบบนุ่ม
                left: 300,behavior: "smooth"
            });
        };

        // getElementById("newBookLeft") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("newBookLeft").onclick = function(){
            // getElementById("newBookList") = หา Element จาก id ที่เราตั้งใน HTML
            // scrollBy() = เลื่อนตำแหน่ง Scroll ของรายการหนังสือ
            document.getElementById("newBookList").scrollBy({
                // left บวก=เลื่อนไปขวา, ติดลบ=ซ้าย; smooth = เลื่อนแบบนุ่ม
                left: -300,behavior: "smooth"
            });
        };

        // getElementById("newBookRight") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("newBookRight").onclick = function(){
            // getElementById("newBookList") = หา Element จาก id ที่เราตั้งใน HTML
            // scrollBy() = เลื่อนตำแหน่ง Scroll ของรายการหนังสือ
            document.getElementById("newBookList").scrollBy({
                // left บวก=เลื่อนไปขวา, ติดลบ=ซ้าย; smooth = เลื่อนแบบนุ่ม
                left: 300,behavior: "smooth"
            });
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 6/22 : books.php
หน้าที่: หน้ารวมหนังสือ ค้นหาคำและกรองหมวดหมู่
เชื่อม: include conn.php; รับ GET จาก index.php/ตัวเอง; คลิกหนังสือ → book_detail.php
FLOW จำ: GET search/category → JOIN books+categories → LIKE/Filter → Query → แสดง Card
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $category_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหมวด
    $category_id = 0;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }
    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["category_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["category_id"])){
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $category_id = intval($_GET["category_id"]);
    }
    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
    // WHERE 1=เงื่อนไขที่จริงเสมอ ใช้เพื่อให้ต่อ AND เพิ่มทีหลังง่าย
    // $sql_book = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_book = "select books.*, categories.category_name 
                 from books 
                 inner join categories on books.category_id = categories.category_id 
                 where 1";
    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);
        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_book .= " and (books.book_title like '%$search_sql%' or books.author_name like '%$search_sql%' or categories.category_name like '%$search_sql%')";
    }
    if($category_id > 0){
        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_book .= " and books.category_id = $category_id";
    }

    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_book .= " order by books.book_id desc";
    // $result_book = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_book = mysqli_query($conn,$sql_book);
    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // IS NULL=ตรวจว่า Field ไม่มีค่า; ใน categories ใช้แยกหมวดหลัก
    // $sql_main_category = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_main_category = "select * from categories where parent_id is null order by category_id";
    // $result_main_category = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_main_category = mysqli_query($conn,$sql_main_category);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>หนังสือ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
        /* .book-card = Class ที่เราตั้งเองสำหรับ Card หนังสือ */
        .book-card{
           height: 100%;
        }
        /* .book-card = Class ที่เราตั้งเองสำหรับ Card หนังสือ */
        .book-card img{
           height: 300px;
           /* object-fit:cover = ให้รูปเต็มกรอบ รักษาสัดส่วนและยอมตัดส่วนเกิน */
           object-fit: cover;
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>
           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>
           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>
           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    <div class="container mt-5 mb-5">
        <h2 class="fw-bold mb-4">
            หนังสือทั้งหมด
        </h2>
        <!-- form ส่งข้อมูลไป books.php แบบ GET -->
        <form action="books.php" method="get" class="row g-2 mb-4">
            <div class="col-md-6">
                <input type="text"
                    name="search"
                    class="form-control"
                    placeholder="ค้นหาชื่อหนังสือ ผู้แต่ง หรือหมวดหมู่"
                    value="<?php echo htmlspecialchars($search); ?>">
            </div>
            <div class="col-md-4">
                <!-- name="category_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_GET["category_id"] -->
                <select name="category_id" class="form-select">
                    <option value="0">
                        ทุกหมวดหมู่
                    </option>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($main_category = mysqli_fetch_assoc($result_main_category)){
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<optgroup label="' . $main_category["category_name"] . '">';

                            // $main_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหมวดหลัก
                            $main_id = $main_category["category_id"];

                            // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                            // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
                            // $sql_sub_category = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                            $sql_sub_category = "select * from categories where parent_id = $main_id order by category_id";
                            // $result_sub_category = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                            // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                            $result_sub_category = mysqli_query($conn,$sql_sub_category);

                            // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                            // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                            while($sub_category = mysqli_fetch_assoc($result_sub_category)){
                              // == = เปรียบเทียบว่าเท่ากันหรือไม่
                              if($category_id == $sub_category["category_id"]){
                                   // $selected = ตัวแปรที่เราสร้างเอง ใช้เก็บค่า selected สำหรับ option
                                   $selected = "selected";
                              // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                              }else{
                                   $selected = "";
                              }
                              // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                              echo '<option value="' . $sub_category["category_id"] . '" ' . $selected . '>';
                              // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                              echo $sub_category["category_name"];
                              // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                              echo '</option>';
                            }
                              // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                              echo '</optgroup>';
                        }
                    ?>
                </select>
                
            </div>
            <div class="col-md-2">
                <button type="submit" class="btn btn-dark w-100">
                    <i class="fa-solid fa-magnifying-glass"></i>
                    ค้นหา
                </button>
            </div>
        </form>
        <div class="row g-4">
            <?php
                // mysqli_num_rows() = Function ของ MySQLi ใช้นับว่าผล SELECT มีทั้งหมดกี่แถว
                if(mysqli_num_rows($result_book) > 0){
                    // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                    // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                    while($book = mysqli_fetch_assoc($result_book)){
            ?>
                <div class="col-6 col-md-4 col-lg-3">
                    <div class="card book-card">
                        <a href="book_detail.php?book_id=<?php echo $book["book_id"]; ?>">
                            <img src="./uploads/books/<?php echo $book["cover_image"]; ?>"
                                class="card-img-top"
                                alt="<?php echo $book["book_title"]; ?>">
                        </a>
                        <div class="card-body">
                            <h5 class="card-title fw-bold">
                                <?php echo $book["book_title"]; ?>
                            </h5>
                            <p class="text-muted mb-1">
                                <?php echo $book["author_name"]; ?>
                            </p>
                            <p class="mb-1">
                                หมวดหมู่ : <?php echo $book["category_name"]; ?>
                            </p>
                            <p class="fw-bold mb-3">
                                <?php echo $book["price"]; ?> บาท
                            </p>
                            <a href="book_detail.php?book_id=<?php echo $book["book_id"]; ?>"
                                class="btn btn-warning w-100">
                                ดูรายละเอียด
                            </a>
                        </div>
                    </div>
                </div>
            <?php
                    }
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<p>ไม่พบหนังสือที่ค้นหา</p>';
                }
            ?>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 7/22 : book_detail.php
หน้าที่: แสดงรายละเอียดหนังสือ 1 เล่มและส่งหนังสือเข้าตะกร้า
เชื่อม: include conn.php; รับ book_id จาก index/books; Form POST book_id+quantity → cart.php
FLOW จำ: GET book_id → SELECT book → แสดง → POST ไป cart.php
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";
     // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["book_id"] = ค่าจาก URL/Form method="get"
     if(isset($_GET["book_id"])){
        // $book_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหนังสือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_GET["book_id"]);
        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_book = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_book = "select * from books where book_id = $book_id";
        // $result_book = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_book = mysqli_query($conn, $sql_book);
        // $book_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหนังสือหน้า Detail
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $book_detail = mysqli_fetch_assoc($result_book);
     }
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>รายละเอียดหนังสือ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>

           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>

           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>

           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    
    <div class="container mt-5 mb-5">
        <div class="row">
            <div class="col-md-4 text-center">
                <img
                    src="./uploads/books/<?php echo $book_detail["cover_image"]; ?>" class="img-fluid" style="max-height: 650px;">
            </div>
            <div class="col-md-8 ps-md-5">
                <h2 class="fw-bold"><?php echo $book_detail["book_title"]; ?></h2>
                <h5 class="text-muted">ผู้แต่ง : <?php echo $book_detail["author_name"]; ?></h5>
                <h4 class="fw-bold"><?php echo $book_detail["price"]; ?> บาท</h4>

                <hr class="my-4">

                <h3 class="fw-bold">รายละเอียดหนังสือ</h3>
                <p class="fs-4"><?php echo $book_detail["description"]; ?></p>

                <hr class="my-4">

                <h3 class="fw-bold">ข้อมูลหนังสือ</h3>
                <p class="fs-5"><strong>วันที่วางจำหน่าย :</strong> <?php echo $book_detail["release_date"]; ?></p>
                <p class="fs-5"><strong>จำนวนคงเหลือ :</strong> <?php echo $book_detail["stock"]; ?> เล่ม</p>

                <!-- form ส่งข้อมูลไป cart.php แบบ POST -->
                <form action="cart.php" method="post" class="mt-4">
                    <!-- name="book_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["book_id"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="book_id" value="<?php echo $book_detail["book_id"]; ?>">
                    <label class="fw-bold">จำนวน</label>
                    <!-- name="quantity" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["quantity"] -->
                    <input type="number" name="quantity" value="1" min="1" max="<?php echo $book_detail["stock"]; ?>" class="form-control mt-2" style="width: 120px;">
                    <button type="submit" class="btn btn-warning mt-3">
                        <i class="fa-solid fa-cart-shopping"></i>
                        เพิ่มลงตะกร้า
                    </button>
                </form>
            </div>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
        
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 8/22 : cart.php
หน้าที่: เก็บ/แสดง/แก้/ลบตะกร้าใน Session และคำนวณยอดรวม
เชื่อม: include conn.php; รับ POST จาก book_detail.php/ตัวเอง; ไป checkout.php
FLOW จำ: Session cart → เพิ่ม/แก้/ลบ → Query หนังสือ → subtotal/total → checkout
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["remove_id"] = ค่าจาก Form method="post"
    if(isset($_POST["remove_id"])){
        // $remove_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID หนังสือที่จะลบจาก Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $remove_id = intval($_POST["remove_id"]);
        // unset() = ลบตัวแปร/สมาชิกที่ระบุ; ตรงนี้ใช้ลบข้อมูลใน Cart/Session
        unset($_SESSION["cart"][$remove_id]);
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["update_id"] = ค่าจาก Form method="post"
    if(isset($_POST["update_id"])){
        // $update_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID หนังสือที่จะอัปเดตใน Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $update_id = intval($_POST["update_id"]);
        // $update_quantity = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวนใหม่ใน Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $update_quantity = intval($_POST["update_quantity"]);
        // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
        $_SESSION["cart"][$update_id] = $update_quantity;
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["book_id"] = ค่าจาก Form method="post"
    if(isset($_POST["book_id"])){
        // $book_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหนังสือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_POST["book_id"]);
        // $quantity = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวน
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $quantity = intval($_POST["quantity"]);
        if(!isset($_SESSION["cart"])){
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"] = array();
        }
        if(isset($_SESSION["cart"][$book_id])){
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"][$book_id] += $quantity;
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"][$book_id] = $quantity;
        }
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>ตะกร้าสินค้า</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>

           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>

           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>

           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div>
    <div class="container-fluid mt-5 mb-5 ms-3">
        <h2 class="fw-bold mb-4">
            ตะกร้าสินค้า
        </h2>
        <?php 
            if(isset($_SESSION["cart"]) && count($_SESSION["cart"]) > 0){
                // $total_price = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดรวม
                $total_price = 0;
                // foreach = Loop สำหรับวนสมาชิก Array; => แยก Key ด้านซ้ายกับ Value ด้านขวา
                foreach($_SESSION["cart"] as $cart_book_id => $cart_quantity){
                    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                    // $sql_cart = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                    $sql_cart = "select * from books where book_id = $cart_book_id";
                    // $result_cart = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                    $result_cart = mysqli_query($conn,$sql_cart);
                    // $cart_book = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหนังสือใน Cart
                    // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                    $cart_book = mysqli_fetch_assoc($result_cart);
                    // $subtotal = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดย่อย
                    $subtotal = $cart_book["price"] * $cart_quantity;
                    $total_price += $subtotal;

                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="d-flex align-items-center gap-4">';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<img src="./uploads/books/' . $cart_book["cover_image"] . '" width="100" height="140" style="object-fit: cover;">';
                      // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                      echo '<div>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<p><strong>หนังสือ :</strong> ' . $cart_book["book_title"] . '</p>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<p><strong>ราคา :</strong> ' . $cart_book["price"] . ' บาท</p>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<p><strong>จำนวน :</strong> ' . $cart_quantity . ' เล่ม</p>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<p><strong>รวม :</strong> ' . $subtotal . ' บาท</p>';

                        // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
                        // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
                        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
                        if(isset($_GET["edit_id"]) && intval($_GET["edit_id"]) == $cart_book_id){
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<form action="cart.php" method="post" class="d-flex align-items-center gap-2 mb-2">';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<input type="hidden" name="update_id" value="' . $cart_book_id . '">';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<strong>จำนวนใหม่ :</strong>';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<input type="number" name="update_quantity" value="' . $cart_quantity . '" min="1" max="' . $cart_book["stock"] . '" class="form-control" style="width:80px;">';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<button type="submit" class="btn btn-success btn-sm">บันทึก</button>';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<a href="cart.php" class="btn btn-secondary btn-sm">ยกเลิก</a>';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '</form>';
                        }

                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<div class="d-flex gap-2">';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<a href="cart.php?edit_id=' . $cart_book_id . '" class="btn btn-success btn-sm">
                                <i class="fa-solid fa-pen"></i> แก้ไข
                              </a>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<form action="cart.php" method="post">';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<input type="hidden" name="remove_id" value="' . $cart_book_id . '">';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '<button type="submit" class="btn btn-danger btn-sm">
                                <i class="fa-solid fa-trash"></i> ลบ
                              </button>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '</form>';
                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                        echo '</div>';
                      // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                      echo '</div>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '</div>';
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<hr style="width: 95%;">';
                }

                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<h3 class="fw-bold">ยอดรวมทั้งหมด : ' . $total_price . ' บาท</h3>';
                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<a href="checkout.php" class="btn btn-warning mt-3">
                        <i class="fa-solid fa-bag-shopping"></i> สั่งซื้อ
                      </a>';

            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<p>ยังไม่มีสินค้าในตะกร้า</p>';
            }
        ?>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 9/22 : cart_exc.php
หน้าที่: ไฟล์ประมวลผลตะกร้าแบบ PHP ล้วนแล้วกลับ cart.php
เชื่อม: include conn.php; รับ POST → แก้ $_SESSION[cart] → cart.php
FLOW จำ: remove/update/add → Session cart → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["remove_id"] = ค่าจาก Form method="post"
    if(isset($_POST["remove_id"])){
        // $remove_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID หนังสือที่จะลบจาก Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $remove_id = intval($_POST["remove_id"]);
        // unset() = ลบตัวแปร/สมาชิกที่ระบุ; ตรงนี้ใช้ลบข้อมูลใน Cart/Session
        unset($_SESSION["cart"][$remove_id]);
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["update_id"] = ค่าจาก Form method="post"
    if(isset($_POST["update_id"])){
        // $update_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID หนังสือที่จะอัปเดตใน Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $update_id = intval($_POST["update_id"]);
        // $update_quantity = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวนใหม่ใน Cart
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $update_quantity = intval($_POST["update_quantity"]);
        // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
        $_SESSION["cart"][$update_id] = $update_quantity;
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["book_id"] = ค่าจาก Form method="post"
    if(isset($_POST["book_id"])){
        // $book_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหนังสือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_POST["book_id"]);
        // $quantity = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวน
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $quantity = intval($_POST["quantity"]);
        if(!isset($_SESSION["cart"])){
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"] = array();
        }
        if(isset($_SESSION["cart"][$book_id])){
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"][$book_id] += $quantity;
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // "cart" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
            $_SESSION["cart"][$book_id] = $quantity;
        }
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
?>

==============================================================================================================
ไฟล์ 10/22 : checkout.php
หน้าที่: เปลี่ยนตะกร้าใน Session เป็นคำสั่งซื้อจริงใน Database
เชื่อม: include conn.php; ต้อง Login; รับ Cart; Upload สลิป; สำเร็จ → my_orders.php
FLOW จำ: เช็ก Login+Cart → total → INSERT orders → insert_id → INSERT details → UPDATE stock → ล้าง cart
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";
     // เช็ก Login: ถ้า Session ไม่มี user_id แปลว่ายังไม่ Login
     if(!isset($_SESSION["user_id"])){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // เช็ก Cart: ถ้ายังไม่มี cart หรือ count() นับได้ 0 ให้กลับ cart.php
    if(!isset($_SESSION["cart"]) || count($_SESSION["cart"]) == 0){
        // header("Location: ...") = Redirect Browser ไป cart.php
        header("Location: cart.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $message = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อความแจ้ง Error/สถานะ
    $message = "";
    // $total_price = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดรวม
    $total_price = 0;

    // foreach = Loop สำหรับวนสมาชิก Array; => แยก Key ด้านซ้ายกับ Value ด้านขวา
    foreach($_SESSION["cart"] as $cart_book_id => $cart_quantity){
        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_book = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_book = "select * from books where book_id = $cart_book_id";
        // $result_book = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_book = mysqli_query($conn,$sql_book);
        // $book = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหนังสือ 1 แถว
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $book = mysqli_fetch_assoc($result_book);

        // $subtotal = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดย่อย
        $subtotal = $book["price"] * $cart_quantity;
        $total_price += $subtotal;
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["checkout"] = ค่าจาก Form method="post"
    if(isset($_POST["checkout"])){
        // $user_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสผู้ใช้
        // "user_id" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
        $user_id = $_SESSION["user_id"];
        // $shipping_address = ตัวแปรที่เราสร้างเอง ใช้เก็บที่อยู่จัดส่ง
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $shipping_address = mysqli_real_escape_string($conn,$_POST["shipping_address"]);
        // $payment_method = ตัวแปรที่เราสร้างเอง ใช้เก็บวิธีชำระเงิน
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $payment_method = mysqli_real_escape_string($conn,$_POST["payment_method"]);
        // $payment_slip = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อไฟล์สลิป
        $payment_slip = "";

        // == = เปรียบเทียบว่าเท่ากันหรือไม่
        if($payment_method == "โอนเงิน"){
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_FILES["payment_slip"] = ข้อมูลไฟล์ Upload
            // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
            if(isset($_FILES["payment_slip"]) && $_FILES["payment_slip"]["name"] != ""){
                // basename() = เอาเฉพาะชื่อไฟล์; time() ด้านหน้าใช้ช่วยให้ชื่อไฟล์ไม่ชนกันง่าย
                $payment_slip = time() . "_" . basename($_FILES["payment_slip"]["name"]);
                // $file_path = ตัวแปรที่เราสร้างเอง ใช้เก็บตำแหน่งไฟล์ปลายทาง
                $file_path = "./uploads/payments/" . $payment_slip;
                // move_uploaded_file() = ย้ายไฟล์จากตำแหน่งชั่วคราว tmp_name ไปโฟลเดอร์จริงของระบบ
                move_uploaded_file($_FILES["payment_slip"]["tmp_name"],$file_path);
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
                $message = "กรุณาแนบสลิปการชำระเงิน";
            }
            // $order_status = ตัวแปรที่เราสร้างเอง ใช้เก็บสถานะคำสั่งซื้อ
            $order_status = "รอตรวจสอบ";
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            $payment_slip = "";
            $order_status = "กำลังเตรียมสินค้า";
        }
        // == = เปรียบเทียบว่าเท่ากันหรือไม่
        if($message == ""){
            // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
            // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
            // $sql_order = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
            $sql_order = "insert into orders (user_id,total_amount,shipping_address,payment_method,payment_slip,order_status) values ($user_id,$total_price,'$shipping_address','$payment_method','$payment_slip','$order_status')";
            // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
            mysqli_query($conn,$sql_order);
            // $order_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสคำสั่งซื้อ
            // mysqli_insert_id() = เอา AUTO_INCREMENT ID ของ Record ที่เพิ่ง INSERT; ตรงนี้ใช้เอา order_id ใหม่
            $order_id = mysqli_insert_id($conn);

            // foreach = Loop สำหรับวนสมาชิก Array; => แยก Key ด้านซ้ายกับ Value ด้านขวา
            foreach($_SESSION["cart"] as $cart_book_id => $cart_quantity){
                // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                $sql_book = "select * from books where book_id = $cart_book_id";
                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                $result_book = mysqli_query($conn,$sql_book);
                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                $book = mysqli_fetch_assoc($result_book);

                // $unit_price = ตัวแปรที่เราสร้างเอง ใช้เก็บราคาต่อหน่วย
                $unit_price = $book["price"];
                $subtotal = $unit_price * $cart_quantity;

                // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
                // $sql_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
                $sql_detail = "insert into order_details (order_id,book_id,quantity,unit_price,subtotal) values ($order_id,$cart_book_id,$cart_quantity,$unit_price,$subtotal)";
                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                mysqli_query($conn,$sql_detail);
                // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
                // $sql_stock = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
                $sql_stock = "update books set stock = stock - $cart_quantity where book_id = $cart_book_id";
                // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                mysqli_query($conn,$sql_stock);
            }
            // unset() = ลบตัวแปร/สมาชิกที่ระบุ; ตรงนี้ใช้ลบข้อมูลใน Cart/Session
            unset($_SESSION["cart"]);
            // header("Location: ...") = Redirect Browser ไป my_orders.php?success=1
            header("Location: my_orders.php?success=1");
            // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
            exit();
        }
    }
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>ยืนยันการสั่งซื้อ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>

           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>

           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>

           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div> 
    <div class="container mt-5 mb-5">
        <h2 class="fw-bold mb-4"> ยืนยันการสั่งซื้อ </h2>
        <?php
            // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
            if($message != ""){
                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<div class="alert alert-danger">' . $message . '</div>';
            }
        ?>
        <div class="row">
            <div class="col-md-7">
                <!-- form ส่งข้อมูลไป checkout.php แบบ POST; multipart/form-data = จำเป็นเมื่อ Upload ไฟล์ -->
                <form action="checkout.php" method="post" enctype="multipart/form-data">
                    <label class="fw-bold mb-2">ที่อยู่จัดส่ง</label>
                    <!-- name="shipping_address" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["shipping_address"] -->
                    <textarea name="shipping_address" class="form-control mb-3" rows="4" required></textarea>

                    <label class="fw-bold mb-2">วิธีชำระเงิน</label>
                    <!-- name="payment_method" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["payment_method"] -->
                    <select name="payment_method" class="form-select mb-3" required>
                        <option value="">-- เลือกวิธีชำระเงิน --</option>
                        <option value="โอนเงิน">โอนเงิน</option>
                        <option value="เก็บเงินปลายทาง">เก็บเงินปลายทาง</option>
                    </select>

                    <label class="fw-bold mb-2">สลิปการชำระเงิน</label>
                    <!-- name="payment_slip" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["payment_slip"] -->
                    <!-- type="file" = ช่องเลือกไฟล์; PHP รับข้อมูลไฟล์ด้วย $_FILES -->
                    <input type="file" name="payment_slip" class="form-control mb-3" accept=".jpg,.jpeg,.png">

                    <!-- name="checkout" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["checkout"] -->
                    <!-- type="submit" = กดแล้วส่ง Form ตาม action/method -->
                    <button type="submit" name="checkout" class="btn btn-success">
                        <i class="fa-solid fa-check"></i> ยืนยันการสั่งซื้อ </button>
                    <a href="cart.php" class="btn btn-secondary ms-2"> กลับไปตะกร้า </a>
                </form>
            </div>
            <div class="col-md-5">
                <div class="border rounded p-3">
                    <h4 class="fw-bold">สรุปคำสั่งซื้อ</h4>
                    <hr>
                    <?php
                        // foreach = Loop สำหรับวนสมาชิก Array; => แยก Key ด้านซ้ายกับ Value ด้านขวา
                        foreach($_SESSION["cart"] as $cart_book_id => $cart_quantity){
                            // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                            // $sql_cart = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                            $sql_cart = "select * from books where book_id = $cart_book_id";
                            // $result_cart = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                            // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                            $result_cart = mysqli_query($conn,$sql_cart);
                            // $cart_book = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหนังสือใน Cart
                            // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                            $cart_book = mysqli_fetch_assoc($result_cart);

                            $subtotal = $cart_book["price"] * $cart_quantity;

                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '<p>';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo $cart_book["book_title"];
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo ' x ' . $cart_quantity;
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo ' = ' . $subtotal . ' บาท';
                            // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                            echo '</p>';
                        }
                    ?>
                    <hr>
                    <h4 class="fw-bold"> ยอดรวม : <?php echo $total_price; ?> บาท </h4>
                </div>
            </div>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
        
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 11/22 : my_orders.php
หน้าที่: แสดงประวัติคำสั่งซื้อของ User ที่ Login อยู่
เชื่อม: include conn.php; ถ้าไม่ Login → login.php; รับ success จาก checkout.php
FLOW จำ: Session user_id → SELECT orders → JOIN details+books → แสดง
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็ก Login: ถ้า Session ไม่มี user_id แปลว่ายังไม่ Login
    if(!isset($_SESSION["user_id"])){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $user_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสผู้ใช้
    // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
    // "user_id" = Key ที่เราเป็นคนตั้งเองใน Session; หน้าอื่นต้องใช้ชื่อเดียวกันจึงอ่านค่าต่อได้
    $user_id = intval($_SESSION["user_id"]);

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // $sql_order = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_order = "select * from orders where user_id = $user_id order by order_id desc";
    // $result_order = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_order = mysqli_query($conn,$sql_order);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>ยืนยันการสั่งซื้อ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* .sidebar = Class ที่เราตั้งเอง; CSS ชุดนี้กำหนด Sidebar และซ่อนไว้ทางซ้าย */
        .sidebar{
           width: 250px;
           position: fixed;
           top: 66px;
           height: calc(100vh - 66px);
           left: -250px;
           background-color: white;
           border-right: 1px solid #333;
           transition: 0.3s;
           z-index: 1000;
        }

        /* .sidebar.active = ตอน JS เพิ่ม active จะเปลี่ยน left เป็น 0 เพื่อแสดง Sidebar */
        .sidebar.active{
           left: 0; 
        }
    </style>
</head>

<body class="d-flex flex-column min-vh-100">
  <!-- Navbar: เมนูด้านบน เชื่อมหน้าแรก, Login/Register/Logout และ Cart -->
  <nav class = "navbar bg-white border-bottom border-dark sticky-top ">
    <div class ="container-fluid" >

        <!-- id="sidebarToggle" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
        <button class="btn btn-dark me-2" id="sidebarToggle" type="button">
            <i class="fa-solid fa-bars"></i>
        </button>

        <a class="navbar-brand me-auto" href="index.php">
        <img src="./img/logo.png" alt="logo" height= "50">
        </a>

        <?php
            // เช็ก Session user_id เพื่อเลือกว่าจะโชว์ชื่อ+Logout หรือ Login+Register
            if(isset($_SESSION["user_id"])){
        ?>
            <span class="me-3">
                <i class="fa-solid fa-user"></i>
                <?php echo $_SESSION["full_name"]; ?>
            </span>
            <a href="logout.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-from-bracket"></i>
                ออกจากระบบ
            </a>
        <?php
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
        ?>
            <a href="login.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-right-to-bracket"></i>
                เข้าสู่ระบบ
            </a>
            <a href="register.php" class="btn btn-outline-dark me-2">
                <i class="fa-solid fa-user-plus"></i>
                สมัครสมาชิก
            </a>
        <?php
            }
        ?>

        <a href="cart.php" class="btn btn-warning ">
            <i class="fa-solid fa-cart-shopping"></i>
            ตะกร้า
        </a>

    </div>
  </nav> 
    <!-- sidebar = Class ที่เราตั้งเอง; sidebarMenu = ID ที่ JS ใช้เปิด/ปิดเมนู -->
    <!-- id="sidebarMenu" = ชื่อที่เราตั้งเอง; JavaScript ต้องเรียก ID นี้ให้ตรง -->
    <div class="sidebar" id="sidebarMenu">
        <div class ="p-4">
           <h6 class="text-muted fw-bold">เมนูหลัก</h6>
           <hr>

           <a href="index.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-house me-2"></i>
            หน้าแรก
           </a>

           <a href="books.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-book me-2"></i>
            หนังสือ
           </a>

           <a href="my_orders.php" class="nav-link text-dark p-2">
            <i class="fa-solid fa-clock-rotate-left me-2"></i>
            ประวัติคำสั่งซื้อ
           </a>
        </div>
    </div> 
    <div class="container mt-5 mb-5">
        <h2 class="fw-bold mb-4"> ประวัติคำสั่งซื้อ </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["success"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["success"])){
                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<div class="alert alert-success">
                        <i class="fa-solid fa-circle-check"></i> สั่งซื้อสำเร็จ
                      </div>';
            }
            // mysqli_num_rows() = Function ของ MySQLi ใช้นับว่าผล SELECT มีทั้งหมดกี่แถว
            if(mysqli_num_rows($result_order) > 0){
                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                while($order = mysqli_fetch_assoc($result_order)){
                    // $order_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสคำสั่งซื้อ
                    $order_id = $order["order_id"];
        ?>
                    <div class="border rounded p-3 mb-4">
                        <div class="row">
                            <div class="col-md-6">
                                <h4 class="fw-bold">
                                    คำสั่งซื้อ #<?php echo $order["order_id"]; ?>
                                </h4>
                                <p class="mb-1">
                                    <strong>วันที่ :</strong>
                                    <?php echo $order["created_at"]; ?>
                                </p>
                                <p class="mb-1">
                                    <strong>วิธีชำระเงิน :</strong>
                                    <?php echo $order["payment_method"]; ?>
                                </p>
                                <p class="mb-1">
                                    <strong>สถานะ :</strong>
                                    <?php echo $order["order_status"]; ?>
                                </p>
                            </div>
                            <div class="col-md-6">
                                <p class="mb-1">
                                    <strong>ที่อยู่จัดส่ง :</strong>
                                    <?php echo $order["shipping_address"]; ?>
                                </p>
                                <h5 class="fw-bold mt-2">
                                    ยอดรวม : <?php echo $order["total_amount"]; ?> บาท
                                </h5>
                            </div>
                        </div>
                        <hr>
                        <h5 class="fw-bold"> รายการหนังสือ </h5>
                        <?php
                            // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
                            // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
                            // $sql_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
                            $sql_detail = "select order_details.*, books.book_title from order_details inner join books on order_details.book_id = books.book_id where order_details.order_id = $order_id";
                            // $result_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
                            // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
                            $result_detail = mysqli_query($conn,$sql_detail);

                            // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                            // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                            while($detail = mysqli_fetch_assoc($result_detail)){
                        ?>
                            <div class="d-flex justify-content-between border-bottom py-2">
                                <div>
                                    <?php echo $detail["book_title"]; ?>
                                    x <?php echo $detail["quantity"]; ?> เล่ม
                                </div>
                                <div>
                                    <?php echo $detail["subtotal"]; ?> บาท
                                </div>
                            </div>
                        <?php
                            }
                        ?>
                    </div>
        <?php
                }
            // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
            }else{
                // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                echo '<p>ยังไม่มีประวัติคำสั่งซื้อ</p>';
            }
        ?>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
        
    </footer>
    <script>
        // getElementById("sidebarToggle") = หา Element จาก id ที่เราตั้งใน HTML
        document.getElementById("sidebarToggle").onclick = function(){
            // getElementById("sidebarMenu") = หา Element จาก id ที่เราตั้งใน HTML
            document.getElementById("sidebarMenu")
            // classList.toggle("active") = สลับ Class active เพื่อเปิด/ปิด Sidebar ร่วมกับ CSS .sidebar.active
            .classList.toggle("active");
        };
    </script>
</body>

</html>

==============================================================================================================
ไฟล์ 12/22 : book.php
หน้าที่: หน้า Admin จัดการหนังสือ: เพิ่ม/ค้นหา/แก้/ลบ/แสดง
เชื่อม: include conn.php; ไม่ใช่ Admin → login.php; Form → book_exc.php; เมนู → category/user/order/order_detail
FLOW จำ: Admin guard → Search/Edit → Query → Form/Table → book_exc.php
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $edit_book = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหนังสือที่โหลดมาแก้
    $edit_book = NULL;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }
    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["edit_id"])){
        // $edit_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID Record ที่จะโหลดมาแก้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $edit_id = intval($_GET["edit_id"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_edit = "select * from books where book_id = $edit_id";
        // $result_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_edit = mysqli_query($conn,$sql_edit);
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $edit_book = mysqli_fetch_assoc($result_edit);
    }
    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
    // $sql_book = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_book = "select books.*, categories.category_name
                 from books
                 inner join categories on books.category_id = categories.category_id";
    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);
        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_book .= " where books.book_title like '%$search_sql%'
                       or books.author_name like '%$search_sql%'
                       or categories.category_name like '%$search_sql%'";
    }
    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_book .= " order by books.book_id desc";
    // $result_book = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_book = mysqli_query($conn,$sql_book);
    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // IS NOT NULL=ตรวจว่า Field มีค่า; ใน categories ใช้เลือกหมวดย่อย
    // $sql_category = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_category = "select * from categories
                     where parent_id is not null
                     order by category_id";
    // $result_category = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_category = mysqli_query($conn,$sql_category);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>จัดการหนังสือ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>

<body class="d-flex flex-column min-vh-100">

    <!-- Navbar Admin: แสดงชื่อจาก Session, Logout และเมนูไป book/category/user/order/order_detail -->
    <nav class="navbar bg-dark navbar-dark">
        <div class="container-fluid">

            <a href="book.php" class="navbar-brand">
                จัดการร้านหนังสือ
            </a>

            <span class="text-white ms-auto me-3">
                <?php echo $_SESSION["full_name"]; ?>
            </span>

            <a href="logout.php" class="btn btn-outline-light">
                ออกจากระบบ
            </a>

        </div>
    </nav>
    <div class="container mt-4 mb-5">
        <div class="mb-4">
            <a href="book.php" class="btn btn-dark">หนังสือ</a>
            <a href="category.php" class="btn btn-outline-dark">หมวดหมู่</a>
            <a href="user.php" class="btn btn-outline-dark">ผู้ใช้</a>
            <a href="order.php" class="btn btn-outline-dark">คำสั่งซื้อ</a>
            <a href="order_detail.php" class="btn btn-outline-dark">รายละเอียดคำสั่งซื้อ</a>
        </div>
        <h2 class="fw-bold mb-4">
            จัดการหนังสือ
        </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["msg"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["msg"])){

                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "insert"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">เพิ่มหนังสือสำเร็จ</div>';
                }

                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "update"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">แก้ไขหนังสือสำเร็จ</div>';
                }

                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "delete"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">ลบหนังสือสำเร็จ</div>';
                }

                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "error"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-danger">ไม่สามารถลบหนังสือได้</div>';
                }
            }
        ?>
        <div class="border rounded p-3 mb-4">
            <?php
                if($edit_book){
            ?>
                <h4 class="fw-bold mb-3">แก้ไขหนังสือ</h4>
                <!-- form ส่งข้อมูลไป book_exc.php แบบ POST; multipart/form-data = จำเป็นเมื่อ Upload ไฟล์ -->
                <form action="book_exc.php" method="post" enctype="multipart/form-data">
                    <!-- name="chk" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["chk"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="update">
                    <input type="hidden"
                        name="book_id"
                        value="<?php echo $edit_book["book_id"]; ?>">
                    <input type="hidden"
                        name="old_cover"
                        value="<?php echo $edit_book["cover_image"]; ?>">
            <?php
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
            ?>
                <h4 class="fw-bold mb-3">เพิ่มหนังสือ</h4>
                <!-- form ส่งข้อมูลไป book_exc.php แบบ POST; multipart/form-data = จำเป็นเมื่อ Upload ไฟล์ -->
                <form action="book_exc.php" method="post" enctype="multipart/form-data">
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="insert">
            <?php
                }
            ?>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ชื่อหนังสือ</label>
                        <input type="text"
                            name="book_title"
                            class="form-control"
                            value="<?php if($edit_book){ echo $edit_book["book_title"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ผู้แต่ง</label>
                        <input type="text"
                            name="author_name"
                            class="form-control"
                            value="<?php if($edit_book){ echo $edit_book["author_name"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">หมวดหมู่</label>
                        <!-- name="category_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["category_id"] -->
                        <select name="category_id"
                            class="form-select"
                            required>
                            <?php
                                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                                while($category = mysqli_fetch_assoc($result_category)){

                                    // $selected = ตัวแปรที่เราสร้างเอง ใช้เก็บค่า selected สำหรับ option
                                    $selected = "";

                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_book && $edit_book["category_id"] == $category["category_id"]){
                                        $selected = "selected";
                                    }

                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $category["category_id"] . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo $category["category_name"];
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-md-3 mb-3">
                        <label class="fw-bold">ราคา</label>
                        <input type="number"
                            name="price"
                            class="form-control"
                            step="0.01"
                            value="<?php if($edit_book){ echo $edit_book["price"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-3 mb-3">
                        <label class="fw-bold">จำนวน</label>
                        <input type="number"
                            name="stock"
                            class="form-control"
                            value="<?php if($edit_book){ echo $edit_book["stock"]; } ?>"
                            required>

                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">วันที่วางจำหน่าย</label>
                        <input type="date"
                            name="release_date"
                            class="form-control"
                            value="<?php if($edit_book){ echo $edit_book["release_date"]; } ?>">

                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">รูปปก</label>
                        <input type="file"
                            name="cover_image"
                            class="form-control">
                    </div>
                    <div class="col-12 mb-3">
                        <label class="fw-bold">รายละเอียด</label>
                        <!-- name="description" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["description"] -->
                        <textarea name="description"
                            class="form-control"
                            rows="3"><?php if($edit_book){ echo $edit_book["description"]; } ?></textarea>
                    </div>
                </div>
                <?php
                    if($edit_book){
                ?>
                    <button type="submit" class="btn btn-success">
                        บันทึกการแก้ไข
                    </button>
                    <a href="book.php" class="btn btn-secondary">
                        ยกเลิก
                    </a>
                <?php
                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                    }else{
                ?>
                    <button type="submit" class="btn btn-success">
                        เพิ่มหนังสือ
                    </button>
                <?php
                    }
                ?>
            </form>
        </div>
        <!-- form ส่งข้อมูลไป book.php แบบ GET -->
        <form action="book.php" method="get" class="d-flex mb-3">
            <input type="text"
                name="search"
                class="form-control me-2"
                placeholder="ค้นหาชื่อหนังสือ ผู้แต่ง หรือหมวดหมู่"
                value="<?php echo $search; ?>">
            <button type="submit" class="btn btn-dark">
                <i class="fa-solid fa-magnifying-glass"></i>
                ค้นหา
            </button>
        </form>
        <div class="table-responsive">
            <table class="table table-bordered table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>ชื่อหนังสือ</th>
                        <th>ผู้แต่ง</th>
                        <th>หมวดหมู่</th>
                        <th>ราคา</th>
                        <th>จำนวน</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($book = mysqli_fetch_assoc($result_book)){
                    ?>
                        <tr>
                            <td><?php echo $book["book_id"]; ?></td>
                            <td><img src="./uploads/books/<?php echo $book["cover_image"]; ?>" width="60" height="80" style="object-fit: cover;"></td>
                            <td><?php echo $book["book_title"]; ?></td>
                            <td><?php echo $book["author_name"]; ?></td>
                            <td><?php echo $book["category_name"]; ?></td>
                            <td><?php echo $book["price"]; ?></td>
                            <td><?php echo $book["stock"]; ?></td>
                            <td>
                                <a href="book.php?edit_id=<?php echo $book["book_id"]; ?>"
                                    class="btn btn-warning btn-sm">
                                    แก้ไข
                                </a>
                                <a href="book_exc.php?val=<?php echo md5($book["book_id"]); ?>&chk=<?php echo md5("delete"); ?>"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('ต้องการลบหนังสือนี้หรือไม่?')">
                                    ลบ
                                </a>
                            </td>
                        </tr>
                    <?php
                        }
                    ?>
                </tbody>
            </table>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>

</body>

</html>

==============================================================================================================
ไฟล์ 13/22 : book_exc.php
หน้าที่: Processor ของ book.php สำหรับ INSERT/UPDATE/DELETE หนังสือและ Upload ปก
เชื่อม: include conn.php; รับ POST/GET จาก book.php; เสร็จ → book.php?msg=...
FLOW จำ: Admin guard → chk → Upload → INSERT/UPDATE/DELETE → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "insert"){
        // $category_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหมวด
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $category_id = intval($_POST["category_id"]);
        // $book_title = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อหนังสือ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $book_title = mysqli_real_escape_string($conn,$_POST["book_title"]);
        // $author_name = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อผู้แต่ง
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $author_name = mysqli_real_escape_string($conn,$_POST["author_name"]);
        // $price = ตัวแปรที่เราสร้างเอง ใช้เก็บราคา
        $price = $_POST["price"];
        // $stock = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวนคงเหลือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $stock = intval($_POST["stock"]);
        // $description = ตัวแปรที่เราสร้างเอง ใช้เก็บรายละเอียด
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $description = mysqli_real_escape_string($conn,$_POST["description"]);
        // $release_date = ตัวแปรที่เราสร้างเอง ใช้เก็บวันที่วางจำหน่าย
        $release_date = $_POST["release_date"];
        // $cover_image = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อไฟล์รูปปก
        $cover_image = "";

        // isset() = เช็กว่ามีค่านี้หรือยัง; $_FILES["cover_image"] = ข้อมูลไฟล์ Upload
        // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
        if(isset($_FILES["cover_image"]) && $_FILES["cover_image"]["name"] != ""){
            // basename() = เอาเฉพาะชื่อไฟล์; time() ด้านหน้าใช้ช่วยให้ชื่อไฟล์ไม่ชนกันง่าย
            $cover_image = time() . "_" . basename($_FILES["cover_image"]["name"]);
            // $file_path = ตัวแปรที่เราสร้างเอง ใช้เก็บตำแหน่งไฟล์ปลายทาง
            $file_path = "./uploads/books/" . $cover_image;
            // move_uploaded_file() = ย้ายไฟล์จากตำแหน่งชั่วคราว tmp_name ไปโฟลเดอร์จริงของระบบ
            move_uploaded_file($_FILES["cover_image"]["tmp_name"],$file_path);
        }
        // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_insert = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
        $sql_insert = "insert into books
        (category_id,book_title,author_name,price,stock,description,cover_image,release_date)
        values
        ($category_id,'$book_title','$author_name',$price,$stock,'$description','$cover_image','$release_date')";

        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        mysqli_query($conn,$sql_insert);

        // header("Location: ...") = Redirect Browser ไป book.php?msg=insert
        header("Location: book.php?msg=insert");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "update"){
        // $book_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหนังสือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_POST["book_id"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $category_id = intval($_POST["category_id"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $book_title = mysqli_real_escape_string($conn,$_POST["book_title"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $author_name = mysqli_real_escape_string($conn,$_POST["author_name"]);
        $price = $_POST["price"];
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $stock = intval($_POST["stock"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $description = mysqli_real_escape_string($conn,$_POST["description"]);
        $release_date = $_POST["release_date"];
        $cover_image = $_POST["old_cover"];

        // isset() = เช็กว่ามีค่านี้หรือยัง; $_FILES["cover_image"] = ข้อมูลไฟล์ Upload
        // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
        if(isset($_FILES["cover_image"]) && $_FILES["cover_image"]["name"] != ""){
            // basename() = เอาเฉพาะชื่อไฟล์; time() ด้านหน้าใช้ช่วยให้ชื่อไฟล์ไม่ชนกันง่าย
            $cover_image = time() . "_" . basename($_FILES["cover_image"]["name"]);
            $file_path = "./uploads/books/" . $cover_image;
            // move_uploaded_file() = ย้ายไฟล์จากตำแหน่งชั่วคราว tmp_name ไปโฟลเดอร์จริงของระบบ
            move_uploaded_file($_FILES["cover_image"]["tmp_name"],$file_path);
        }

        // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_update = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
        $sql_update = "update books set
        category_id = $category_id,
        book_title = '$book_title',
        author_name = '$author_name',
        price = $price,
        stock = $stock,
        description = '$description',
        cover_image = '$cover_image',
        release_date = '$release_date'
        where book_id = $book_id";

        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        mysqli_query($conn,$sql_update);

        // header("Location: ...") = Redirect Browser ไป book.php?msg=update
        header("Location: book.php?msg=update");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["chk"] = ค่าจาก URL/Form method="get"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
    if(isset($_GET["chk"]) && $_GET["chk"] == md5("delete")){
        // $val = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่ส่งมาใช้ระบุ Record ตอนลบ
        $val = $_GET["val"];
        // DELETE FROM=ลบ Record; WHERE=เลือก Record ที่จะลบ
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_delete = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง DELETE สำหรับส่งไป MySQL
        // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
        $sql_delete = "delete from books where md5(book_id) = '$val'";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_delete)){
            // header("Location: ...") = Redirect Browser ไป book.php?msg=delete
            header("Location: book.php?msg=delete");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
           // header("Location: ...") = Redirect Browser ไป book.php?msg=error
           header("Location: book.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // header("Location: ...") = Redirect Browser ไป book.php
    header("Location: book.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 14/22 : category.php
หน้าที่: หน้า Admin จัดการหมวดหมู่และหมวดย่อย
เชื่อม: include conn.php; Form → category_exc.php; เมนู Admin เชื่อมทุกตาราง
FLOW จำ: Admin guard → Search/Edit → Query categories → Form/Table
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $edit_category = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูลหมวดที่โหลดมาแก้
    $edit_category = NULL;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["edit_id"])){
        // $edit_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID Record ที่จะโหลดมาแก้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $edit_id = intval($_GET["edit_id"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_edit = "select * from categories where category_id = $edit_id";
        // $result_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_edit = mysqli_query($conn,$sql_edit);
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $edit_category = mysqli_fetch_assoc($result_edit);
    }

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // LEFT JOIN=เชื่อมตารางโดยยังเก็บทุกแถวฝั่งซ้าย; ON=บอก Field ที่ใช้จับคู่
    // $sql_category = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_category = "select c.*, p.category_name as parent_name
                     from categories c
                     left join categories p on c.parent_id = p.category_id";

    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);

        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_category .= " where c.category_name like '%$search_sql%'
                           or c.description like '%$search_sql%'
                           or p.category_name like '%$search_sql%'";
    }

    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_category .= " order by c.category_id";

    // $result_category = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_category = mysqli_query($conn,$sql_category);

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // IS NULL=ตรวจว่า Field ไม่มีค่า; ใน categories ใช้แยกหมวดหลัก
    // $sql_main = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_main = "select * from categories
                 where parent_id is null
                 order by category_id";

    // $result_main = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_main = mysqli_query($conn,$sql_main);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>จัดการหมวดหมู่</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>

<body class="d-flex flex-column min-vh-100">

    <!-- Navbar Admin: แสดงชื่อจาก Session, Logout และเมนูไป book/category/user/order/order_detail -->
    <nav class="navbar bg-dark navbar-dark">
        <div class="container-fluid">

            <a href="book.php" class="navbar-brand">
                จัดการร้านหนังสือ
            </a>

            <span class="text-white ms-auto me-3">
                <?php echo $_SESSION["full_name"]; ?>
            </span>

            <a href="logout.php" class="btn btn-outline-light">
                ออกจากระบบ
            </a>

        </div>
    </nav>
    <div class="container mt-4 mb-5">
        <div class="mb-4">
            <a href="book.php" class="btn btn-outline-dark">หนังสือ</a>
            <a href="category.php" class="btn btn-dark">หมวดหมู่</a>
            <a href="user.php" class="btn btn-outline-dark">ผู้ใช้</a>
            <a href="order.php" class="btn btn-outline-dark">คำสั่งซื้อ</a>
            <a href="order_detail.php" class="btn btn-outline-dark">รายละเอียดคำสั่งซื้อ</a>
        </div>
        <h2 class="fw-bold mb-4">
            จัดการหมวดหมู่
        </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["msg"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["msg"])){
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "insert"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">เพิ่มหมวดหมู่สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "update"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">แก้ไขหมวดหมู่สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "delete"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">ลบหมวดหมู่สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "error"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-danger">ไม่สามารถลบหมวดหมู่นี้ได้</div>';
                }
            }
        ?>
        <div class="border rounded p-3 mb-4">
            <?php
                if($edit_category){
            ?>
                <h4 class="fw-bold mb-3">แก้ไขหมวดหมู่</h4>
                <!-- form ส่งข้อมูลไป category_exc.php แบบ POST -->
                <form action="category_exc.php" method="post">
                    <!-- name="chk" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["chk"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="update">
                    <!-- name="category_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["category_id"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="category_id" value="<?php echo $edit_category["category_id"]; ?>">
            <?php
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
            ?>
                <h4 class="fw-bold mb-3">เพิ่มหมวดหมู่</h4>
                <!-- form ส่งข้อมูลไป category_exc.php แบบ POST -->
                <form action="category_exc.php" method="post">
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="insert">
            <?php
                }
            ?>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ชื่อหมวดหมู่</label>
                        <input type="text"
                            name="category_name"
                            class="form-control"
                            value="<?php if($edit_category){ echo $edit_category["category_name"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">หมวดหมู่หลัก</label>
                        <!-- name="parent_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["parent_id"] -->
                        <select name="parent_id" class="form-select">
                            <option value="0">
                                เป็นหมวดหมู่หลัก
                            </option>
                            <?php
                                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                                while($main = mysqli_fetch_assoc($result_main)){

                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_category && $edit_category["category_id"] == $main["category_id"]){
                                        continue;
                                    }

                                    // $selected = ตัวแปรที่เราสร้างเอง ใช้เก็บค่า selected สำหรับ option
                                    $selected = "";

                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_category && $edit_category["parent_id"] == $main["category_id"]){
                                        $selected = "selected";
                                    }
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $main["category_id"] . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo $main["category_name"];
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-12 mb-3">
                        <label class="fw-bold">รายละเอียด</label>
                        <!-- name="description" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["description"] -->
                        <textarea name="description"
                            class="form-control"
                            rows="3"><?php if($edit_category){ echo $edit_category["description"]; } ?></textarea>
                    </div>
                </div>
                <?php
                    if($edit_category){
                ?>
                    <button type="submit" class="btn btn-success">
                        บันทึกการแก้ไข
                    </button>
                    <a href="category.php" class="btn btn-secondary">
                        ยกเลิก
                    </a>
                <?php
                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                    }else{
                ?>
                    <button type="submit" class="btn btn-success">
                        เพิ่มหมวดหมู่
                    </button>
                <?php
                    }
                ?>
            </form>
        </div>
        <!-- form ส่งข้อมูลไป category.php แบบ GET -->
        <form action="category.php" method="get" class="d-flex mb-3">
            <input type="text"
                name="search"
                class="form-control me-2"
                placeholder="ค้นหาหมวดหมู่"
                value="<?php echo $search; ?>">
            <button type="submit" class="btn btn-dark">
                <i class="fa-solid fa-magnifying-glass"></i>
                ค้นหา
            </button>
        </form>
        <div class="table-responsive">
            <table class="table table-bordered table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>ชื่อหมวดหมู่</th>
                        <th>หมวดหมู่หลัก</th>
                        <th>รายละเอียด</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($category = mysqli_fetch_assoc($result_category)){
                    ?>
                        <tr>
                            <td><?php echo $category["category_id"]; ?></td>
                            <td><?php echo $category["category_name"]; ?></td>
                            <td>
                                <?php
                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($category["parent_id"] == NULL){
                                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                        echo "หมวดหมู่หลัก";
                                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                                    }else{
                                        // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                        echo $category["parent_name"];
                                    }
                                ?>
                            </td>
                            <td><?php echo $category["description"]; ?></td>
                            <td>
                                <a href="category.php?edit_id=<?php echo $category["category_id"]; ?>"
                                    class="btn btn-warning btn-sm">
                                    แก้ไข
                                </a>
                                <a href="category_exc.php?val=<?php echo md5($category["category_id"]); ?>&chk=<?php echo md5("delete"); ?>"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('ต้องการลบหมวดหมู่นี้หรือไม่?')">
                                    ลบ
                                </a>
                            </td>
                        </tr>
                    <?php
                        }
                    ?>
                </tbody>
            </table>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
</body>

</html>

==============================================================================================================
ไฟล์ 15/22 : category_exc.php
หน้าที่: Processor ของ category.php สำหรับ INSERT/UPDATE/DELETE หมวด
เชื่อม: include conn.php; รับจาก category.php; เสร็จ → category.php?msg=...
FLOW จำ: Admin guard → chk → parent_id → CRUD → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "insert"){

        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $category_name = mysqli_real_escape_string($conn,$_POST["category_name"]);
        // $parent_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหมวดแม่
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $parent_id = intval($_POST["parent_id"]);
        // $description = ตัวแปรที่เราสร้างเอง ใช้เก็บรายละเอียด
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $description = mysqli_real_escape_string($conn,$_POST["description"]);
        // == = เปรียบเทียบว่าเท่ากันหรือไม่
        if($parent_id == 0){
            // $parent_value = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่จะบันทึกใน parent_id
            $parent_value = "NULL";
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            $parent_value = $parent_id;
        }
        // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_insert = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
        $sql_insert = "insert into categories
        (parent_id,category_name,description)
        values
        ($parent_value,'$category_name','$description')";

        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        mysqli_query($conn,$sql_insert);

        // header("Location: ...") = Redirect Browser ไป category.php?msg=insert
        header("Location: category.php?msg=insert");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "update"){

        // $category_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหมวด
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $category_id = intval($_POST["category_id"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $category_name = mysqli_real_escape_string($conn,$_POST["category_name"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $parent_id = intval($_POST["parent_id"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $description = mysqli_real_escape_string($conn,$_POST["description"]);
        // == = เปรียบเทียบว่าเท่ากันหรือไม่
        if($parent_id == 0){
            $parent_value = "NULL";
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            $parent_value = $parent_id;
        }
        // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_update = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
        $sql_update = "update categories set
        parent_id = $parent_value,
        category_name = '$category_name',
        description = '$description'
        where category_id = $category_id";

        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        mysqli_query($conn,$sql_update);

        // header("Location: ...") = Redirect Browser ไป category.php?msg=update
        header("Location: category.php?msg=update");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["chk"] = ค่าจาก URL/Form method="get"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
    if(isset($_GET["chk"]) && $_GET["chk"] == md5("delete")){
        // $val = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่ส่งมาใช้ระบุ Record ตอนลบ
        $val = $_GET["val"];
        // DELETE FROM=ลบ Record; WHERE=เลือก Record ที่จะลบ
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_delete = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง DELETE สำหรับส่งไป MySQL
        $sql_delete = "delete from categories
                       // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
                       where md5(category_id) = '$val'";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_delete)){
            // header("Location: ...") = Redirect Browser ไป category.php?msg=delete
            header("Location: category.php?msg=delete");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป category.php?msg=error
            header("Location: category.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // header("Location: ...") = Redirect Browser ไป category.php
    header("Location: category.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 16/22 : user.php
หน้าที่: หน้า Admin จัดการสมาชิก
เชื่อม: include conn.php; Form → user_exc.php; เมนู Admin เชื่อมทุกตาราง
FLOW จำ: Admin guard → Search/Edit → Query users → Form/Table
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $edit_user = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูล User ที่โหลดมาแก้
    $edit_user = NULL;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["edit_id"])){
        // $edit_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID Record ที่จะโหลดมาแก้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $edit_id = intval($_GET["edit_id"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_edit = "select * from users where user_id = $edit_id";
        // $result_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_edit = mysqli_query($conn,$sql_edit);
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $edit_user = mysqli_fetch_assoc($result_edit);
    }

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // $sql_user = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_user = "select * from users";

    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);

        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_user .= " where full_name like '%$search_sql%'
                       or email like '%$search_sql%'
                       or phone like '%$search_sql%'
                       or role like '%$search_sql%'";
    }

    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_user .= " order by user_id desc";
    // $result_user = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_user = mysqli_query($conn,$sql_user);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>จัดการผู้ใช้</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>

<body class="d-flex flex-column min-vh-100">

    <!-- Navbar Admin: แสดงชื่อจาก Session, Logout และเมนูไป book/category/user/order/order_detail -->
    <nav class="navbar bg-dark navbar-dark">
        <div class="container-fluid">

            <a href="book.php" class="navbar-brand">
                จัดการร้านหนังสือ
            </a>

            <span class="text-white ms-auto me-3">
                <?php echo $_SESSION["full_name"]; ?>
            </span>

            <a href="logout.php" class="btn btn-outline-light">
                ออกจากระบบ
            </a>

        </div>
    </nav>
    <div class="container mt-4 mb-5">
        <div class="mb-4">
            <a href="book.php" class="btn btn-outline-dark">หนังสือ</a>
            <a href="category.php" class="btn btn-outline-dark">หมวดหมู่</a>
            <a href="user.php" class="btn btn-dark">ผู้ใช้</a>
            <a href="order.php" class="btn btn-outline-dark">คำสั่งซื้อ</a>
            <a href="order_detail.php" class="btn btn-outline-dark">รายละเอียดคำสั่งซื้อ</a>
        </div>
        <h2 class="fw-bold mb-4">
            จัดการผู้ใช้
        </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["msg"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["msg"])){
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "insert"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">เพิ่มผู้ใช้สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "update"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">แก้ไขผู้ใช้สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "delete"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">ลบผู้ใช้สำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "error"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-danger">ไม่สามารถดำเนินการได้</div>';
                }
            }
        ?>
        <div class="border rounded p-3 mb-4">
            <?php
                if($edit_user){
            ?>
                <h4 class="fw-bold mb-3">แก้ไขผู้ใช้</h4>
                <!-- form ส่งข้อมูลไป user_exc.php แบบ POST -->
                <form action="user_exc.php" method="post">
                    <!-- name="chk" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["chk"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="update">
                    <!-- name="user_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["user_id"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="user_id" value="<?php echo $edit_user["user_id"]; ?>">
            <?php
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
            ?>
                <h4 class="fw-bold mb-3">เพิ่มผู้ใช้</h4>
                <!-- form ส่งข้อมูลไป user_exc.php แบบ POST -->
                <form action="user_exc.php" method="post">
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="insert">
            <?php
                }
            ?>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ชื่อ - นามสกุล</label>
                        <input type="text"
                            name="full_name"
                            class="form-control"
                            value="<?php if($edit_user){ echo $edit_user["full_name"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">อีเมล</label>
                        <input type="email"
                            name="email"
                            class="form-control"
                            value="<?php if($edit_user){ echo $edit_user["email"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">รหัสผ่าน</label>
                        <input type="text"
                            name="password"
                            class="form-control"
                            value="<?php if($edit_user){ echo $edit_user["password"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">เบอร์โทรศัพท์</label>
                        <input type="text"
                            name="phone"
                            class="form-control"
                            value="<?php if($edit_user){ echo $edit_user["phone"]; } ?>">
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ประเภทผู้ใช้</label>
                        <!-- name="role" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["role"] -->
                        <select name="role" class="form-select">
                            <option value="member"
                                <?php if($edit_user && $edit_user["role"] == "member"){ echo "selected"; } ?>>
                                member
                            </option>
                            <option value="admin"
                                <?php if($edit_user && $edit_user["role"] == "admin"){ echo "selected"; } ?>>
                                admin
                            </option>
                        </select>
                    </div>
                    <div class="col-12 mb-3">
                        <label class="fw-bold">ที่อยู่</label>
                        <textarea name="address"
                            class="form-control"
                            rows="3"><?php if($edit_user){ echo $edit_user["address"]; } ?></textarea>
                    </div>
                </div>
                <?php
                    if($edit_user){
                ?>
                    <button type="submit" class="btn btn-success">
                        บันทึกการแก้ไข
                    </button>
                    <a href="user.php" class="btn btn-secondary">
                        ยกเลิก
                    </a>
                <?php
                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                    }else{
                ?>
                    <button type="submit" class="btn btn-success">
                        เพิ่มผู้ใช้
                    </button>
                <?php
                    }
                ?>
            </form>
        </div>
        <!-- form ส่งข้อมูลไป user.php แบบ GET -->
        <form action="user.php" method="get" class="d-flex mb-3">
            <input type="text"
                name="search"
                class="form-control me-2"
                placeholder="ค้นหาชื่อ อีเมล เบอร์โทร หรือประเภทผู้ใช้"
                value="<?php echo $search; ?>">
            <button type="submit" class="btn btn-dark">
                <i class="fa-solid fa-magnifying-glass"></i>
                ค้นหา
            </button>
        </form>
        <div class="table-responsive">
            <table class="table table-bordered table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>ชื่อ - นามสกุล</th>
                        <th>อีเมล</th>
                        <th>เบอร์โทร</th>
                        <th>ประเภท</th>
                        <th>ที่อยู่</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($user = mysqli_fetch_assoc($result_user)){
                    ?>
                        <tr>
                            <td><?php echo $user["user_id"]; ?></td>
                            <td><?php echo $user["full_name"]; ?></td>
                            <td><?php echo $user["email"]; ?></td>
                            <td><?php echo $user["phone"]; ?></td>
                            <td><?php echo $user["role"]; ?></td>
                            <td><?php echo $user["address"]; ?></td>
                            <td>
                                <a href="user.php?edit_id=<?php echo $user["user_id"]; ?>"
                                    class="btn btn-warning btn-sm">
                                    แก้ไข
                                </a>
                                <a href="user_exc.php?val=<?php echo md5($user["user_id"]); ?>&chk=<?php echo md5("delete"); ?>"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('ต้องการลบผู้ใช้นี้หรือไม่?')">
                                    ลบ
                                </a>
                            </td>
                        </tr>
                    <?php
                        }
                    ?>
                </tbody>
            </table>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
</body>

</html>

==============================================================================================================
ไฟล์ 17/22 : user_exc.php
หน้าที่: Processor ของ user.php สำหรับ INSERT/UPDATE/DELETE users
เชื่อม: include conn.php; รับจาก user.php; เสร็จ → user.php?msg=...
FLOW จำ: Admin guard → chk → CRUD users → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "insert"){

        // $full_name = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อ-นามสกุล
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $full_name = mysqli_real_escape_string($conn,$_POST["full_name"]);
        // $email = ตัวแปรที่เราสร้างเอง ใช้เก็บEmail จาก Form
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $email = mysqli_real_escape_string($conn,$_POST["email"]);
        // $password = ตัวแปรที่เราสร้างเอง ใช้เก็บPassword ตามบริบทของไฟล์
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $password = mysqli_real_escape_string($conn,$_POST["password"]);
        // $phone = ตัวแปรที่เราสร้างเอง ใช้เก็บเบอร์โทร
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $phone = mysqli_real_escape_string($conn,$_POST["phone"]);
        // $address = ตัวแปรที่เราสร้างเอง ใช้เก็บที่อยู่
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $address = mysqli_real_escape_string($conn,$_POST["address"]);
        // $role = ตัวแปรที่เราสร้างเอง ใช้เก็บสิทธิ์ผู้ใช้
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $role = mysqli_real_escape_string($conn,$_POST["role"]);

        // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_insert = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
        $sql_insert = "insert into users
        (full_name,email,password,phone,address,role)
        values
        ('$full_name','$email','$password','$phone','$address','$role')";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_insert)){
            // header("Location: ...") = Redirect Browser ไป user.php?msg=insert
            header("Location: user.php?msg=insert");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป user.php?msg=error
            header("Location: user.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "update"){

        // $user_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสผู้ใช้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $user_id = intval($_POST["user_id"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $full_name = mysqli_real_escape_string($conn,$_POST["full_name"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $email = mysqli_real_escape_string($conn,$_POST["email"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $password = mysqli_real_escape_string($conn,$_POST["password"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $phone = mysqli_real_escape_string($conn,$_POST["phone"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $address = mysqli_real_escape_string($conn,$_POST["address"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $role = mysqli_real_escape_string($conn,$_POST["role"]);

        // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_update = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
        $sql_update = "update users set
        full_name = '$full_name',
        email = '$email',
        password = '$password',
        phone = '$phone',
        address = '$address',
        role = '$role'
        where user_id = $user_id";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_update)){
            // header("Location: ...") = Redirect Browser ไป user.php?msg=update
            header("Location: user.php?msg=update");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป user.php?msg=error
            header("Location: user.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["chk"] = ค่าจาก URL/Form method="get"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
    if(isset($_GET["chk"]) && $_GET["chk"] == md5("delete")){
        // $val = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่ส่งมาใช้ระบุ Record ตอนลบ
        $val = $_GET["val"];
        // DELETE FROM=ลบ Record; WHERE=เลือก Record ที่จะลบ
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_delete = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง DELETE สำหรับส่งไป MySQL
        $sql_delete = "delete from users
                       // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
                       where md5(user_id) = '$val'";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_delete)){
            // header("Location: ...") = Redirect Browser ไป user.php?msg=delete
            header("Location: user.php?msg=delete");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป user.php?msg=error
            header("Location: user.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // header("Location: ...") = Redirect Browser ไป user.php
    header("Location: user.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 18/22 : order.php
หน้าที่: หน้า Admin จัดการคำสั่งซื้อ
เชื่อม: include conn.php; Form → order_exc.php; Query เชื่อม users
FLOW จำ: Admin guard → Search/Edit → JOIN users → Form/Table
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $edit_order = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูล Order ที่โหลดมาแก้
    $edit_order = NULL;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["edit_id"])){
        // $edit_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID Record ที่จะโหลดมาแก้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $edit_id = intval($_GET["edit_id"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_edit = "select * from orders where order_id = $edit_id";
        // $result_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_edit = mysqli_query($conn,$sql_edit);
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $edit_order = mysqli_fetch_assoc($result_edit);
    }

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
    // $sql_order = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_order = "select orders.*, users.full_name
                  from orders
                  inner join users on orders.user_id = users.user_id";

    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);

        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_order .= " where orders.order_id like '%$search_sql%'
                        or users.full_name like '%$search_sql%'
                        or orders.payment_method like '%$search_sql%'
                        or orders.order_status like '%$search_sql%'";
    }

    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_order .= " order by orders.order_id desc";
    // $result_order = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_order = mysqli_query($conn,$sql_order);

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
    // $sql_user = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_user = "select * from users where role = 'member' order by user_id";
    // $result_user = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_user = mysqli_query($conn,$sql_user);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>จัดการคำสั่งซื้อ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>

<body class="d-flex flex-column min-vh-100">

    <!-- Navbar Admin: แสดงชื่อจาก Session, Logout และเมนูไป book/category/user/order/order_detail -->
    <nav class="navbar bg-dark navbar-dark">
        <div class="container-fluid">

            <a href="book.php" class="navbar-brand">
                จัดการร้านหนังสือ
            </a>

            <span class="text-white ms-auto me-3">
                <?php echo $_SESSION["full_name"]; ?>
            </span>

            <a href="logout.php" class="btn btn-outline-light">
                ออกจากระบบ
            </a>

        </div>
    </nav>
    <div class="container mt-4 mb-5">
        <div class="mb-4">
            <a href="book.php" class="btn btn-outline-dark">หนังสือ</a>
            <a href="category.php" class="btn btn-outline-dark">หมวดหมู่</a>
            <a href="user.php" class="btn btn-outline-dark">ผู้ใช้</a>
            <a href="order.php" class="btn btn-dark">คำสั่งซื้อ</a>
            <a href="order_detail.php" class="btn btn-outline-dark">รายละเอียดคำสั่งซื้อ</a>
        </div>
        <h2 class="fw-bold mb-4">
            จัดการคำสั่งซื้อ
        </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["msg"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["msg"])){
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "insert"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">เพิ่มคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "update"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">แก้ไขคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "delete"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">ลบคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "error"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-danger">ไม่สามารถดำเนินการได้</div>';
                }
            }
        ?>
        <div class="border rounded p-3 mb-4">
            <?php
                if($edit_order){
            ?>
                <h4 class="fw-bold mb-3">แก้ไขคำสั่งซื้อ</h4>
                <!-- form ส่งข้อมูลไป order_exc.php แบบ POST -->
                <form action="order_exc.php" method="post">
                    <!-- name="chk" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["chk"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="update">
                    <!-- name="order_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["order_id"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="order_id" value="<?php echo $edit_order["order_id"]; ?>">
            <?php
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
            ?>
                <h4 class="fw-bold mb-3">เพิ่มคำสั่งซื้อ</h4>
                <!-- form ส่งข้อมูลไป order_exc.php แบบ POST -->
                <form action="order_exc.php" method="post">
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="insert">
            <?php
                }
            ?>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ผู้ใช้</label>
                        <!-- name="user_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["user_id"] -->
                        <select name="user_id" class="form-select" required>
                            <?php
                                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                                while($user = mysqli_fetch_assoc($result_user)){
                                    // $selected = ตัวแปรที่เราสร้างเอง ใช้เก็บค่า selected สำหรับ option
                                    $selected = "";
                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_order && $edit_order["user_id"] == $user["user_id"]){
                                        $selected = "selected";
                                    }
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $user["user_id"] . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo $user["full_name"];
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ยอดรวม</label>
                        <input type="number"
                            name="total_amount"
                            class="form-control"
                            step="0.01"
                            value="<?php if($edit_order){ echo $edit_order["total_amount"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">วิธีชำระเงิน</label>
                        <!-- name="payment_method" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["payment_method"] -->
                        <select name="payment_method" class="form-select" required>
                            <option value="โอนเงิน"
                                <?php if($edit_order && $edit_order["payment_method"] == "โอนเงิน"){ echo "selected"; } ?>>
                                โอนเงิน
                            </option>
                            <option value="เก็บเงินปลายทาง"
                                <?php if($edit_order && $edit_order["payment_method"] == "เก็บเงินปลายทาง"){ echo "selected"; } ?>>
                                เก็บเงินปลายทาง
                            </option>
                        </select>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">สถานะคำสั่งซื้อ</label>
                        <select name="order_status" class="form-select" required>
                            <?php
                                $status_list = array(
                                    "รอชำระเงิน",
                                    "รอตรวจสอบ",
                                    "กำลังเตรียมสินค้า",
                                    "จัดส่งแล้ว",
                                    "เสร็จสิ้น",
                                    "ยกเลิก"
                                );
                                // foreach = Loop สำหรับวนสมาชิก Array; => แยก Key ด้านซ้ายกับ Value ด้านขวา
                                foreach($status_list as $status){
                                    $selected = "";
                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_order && $edit_order["order_status"] == $status){
                                        $selected = "selected";
                                    }

                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $status . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo $status;
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ชื่อไฟล์สลิป</label>
                        <input type="text"
                            name="payment_slip"
                            class="form-control"
                            value="<?php if($edit_order){ echo $edit_order["payment_slip"]; } ?>">
                    </div>
                    <div class="col-12 mb-3">
                        <label class="fw-bold">ที่อยู่จัดส่ง</label>
                        <!-- name="shipping_address" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["shipping_address"] -->
                        <textarea name="shipping_address"
                            class="form-control"
                            rows="3"
                            required><?php if($edit_order){ echo $edit_order["shipping_address"]; } ?></textarea>
                    </div>
                </div>
                <?php
                    if($edit_order){
                ?>
                    <button type="submit" class="btn btn-success">
                        บันทึกการแก้ไข
                    </button>
                    <a href="order.php" class="btn btn-secondary">
                        ยกเลิก
                    </a>
                <?php
                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                    }else{
                ?>
                    <button type="submit" class="btn btn-success">
                        เพิ่มคำสั่งซื้อ
                    </button>
                <?php
                    }
                ?>
            </form>
        </div>
        <!-- form ส่งข้อมูลไป order.php แบบ GET -->
        <form action="order.php" method="get" class="d-flex mb-3">
            <input type="text"
                name="search"
                class="form-control me-2"
                placeholder="ค้นหาเลขคำสั่งซื้อ ชื่อผู้ใช้ วิธีชำระ หรือสถานะ"
                value="<?php echo $search; ?>">
            <button type="submit" class="btn btn-dark">
                <i class="fa-solid fa-magnifying-glass"></i>
                ค้นหา
            </button>
        </form>
        <div class="table-responsive">
            <table class="table table-bordered table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>ผู้ใช้</th>
                        <th>ยอดรวม</th>
                        <th>วิธีชำระเงิน</th>
                        <th>สถานะ</th>
                        <th>วันที่</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($order = mysqli_fetch_assoc($result_order)){
                    ?>
                        <tr>
                            <td><?php echo $order["order_id"]; ?></td>
                            <td><?php echo $order["full_name"]; ?></td>
                            <td><?php echo $order["total_amount"]; ?> บาท</td>
                            <td><?php echo $order["payment_method"]; ?></td>
                            <td><?php echo $order["order_status"]; ?></td>
                            <td><?php echo $order["created_at"]; ?></td>
                            <td>
                                <a href="order.php?edit_id=<?php echo $order["order_id"]; ?>"
                                    class="btn btn-warning btn-sm">
                                    แก้ไข
                                </a>
                                <a href="order_exc.php?val=<?php echo md5($order["order_id"]); ?>&chk=<?php echo md5("delete"); ?>"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('ต้องการลบคำสั่งซื้อนี้หรือไม่?')">
                                    ลบ
                                </a>
                            </td>
                        </tr>
                    <?php
                        }
                    ?>
                </tbody>
            </table>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
</body>

</html>

==============================================================================================================
ไฟล์ 19/22 : order_exc.php
หน้าที่: Processor ของ order.php สำหรับ INSERT/UPDATE/DELETE orders
เชื่อม: include conn.php; รับจาก order.php; เสร็จ → order.php?msg=...
FLOW จำ: Admin guard → chk → CRUD orders → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "insert"){

        // $user_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสผู้ใช้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $user_id = intval($_POST["user_id"]);
        // $total_amount = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดรวมคำสั่งซื้อ
        $total_amount = $_POST["total_amount"];
        // $shipping_address = ตัวแปรที่เราสร้างเอง ใช้เก็บที่อยู่จัดส่ง
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $shipping_address = mysqli_real_escape_string($conn,$_POST["shipping_address"]);
        // $payment_method = ตัวแปรที่เราสร้างเอง ใช้เก็บวิธีชำระเงิน
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $payment_method = mysqli_real_escape_string($conn,$_POST["payment_method"]);
        // $payment_slip = ตัวแปรที่เราสร้างเอง ใช้เก็บชื่อไฟล์สลิป
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $payment_slip = mysqli_real_escape_string($conn,$_POST["payment_slip"]);
        // $order_status = ตัวแปรที่เราสร้างเอง ใช้เก็บสถานะคำสั่งซื้อ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $order_status = mysqli_real_escape_string($conn,$_POST["order_status"]);

        // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_insert = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
        $sql_insert = "insert into orders
        (user_id,total_amount,shipping_address,payment_method,payment_slip,order_status)
        values
        ($user_id,$total_amount,'$shipping_address','$payment_method','$payment_slip','$order_status')";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_insert)){
            // header("Location: ...") = Redirect Browser ไป order.php?msg=insert
            header("Location: order.php?msg=insert");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order.php?msg=error
            header("Location: order.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "update"){

        // $order_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสคำสั่งซื้อ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $order_id = intval($_POST["order_id"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $user_id = intval($_POST["user_id"]);
        $total_amount = $_POST["total_amount"];
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $shipping_address = mysqli_real_escape_string($conn,$_POST["shipping_address"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $payment_method = mysqli_real_escape_string($conn,$_POST["payment_method"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $payment_slip = mysqli_real_escape_string($conn,$_POST["payment_slip"]);
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $order_status = mysqli_real_escape_string($conn,$_POST["order_status"]);

        // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_update = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
        $sql_update = "update orders set
        user_id = $user_id,
        total_amount = $total_amount,
        shipping_address = '$shipping_address',
        payment_method = '$payment_method',
        payment_slip = '$payment_slip',
        order_status = '$order_status'
        where order_id = $order_id";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_update)){
            // header("Location: ...") = Redirect Browser ไป order.php?msg=update
            header("Location: order.php?msg=update");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order.php?msg=error
            header("Location: order.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["chk"] = ค่าจาก URL/Form method="get"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
    if(isset($_GET["chk"]) && $_GET["chk"] == md5("delete")){
        // $val = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่ส่งมาใช้ระบุ Record ตอนลบ
        $val = $_GET["val"];
        // DELETE FROM=ลบ Record; WHERE=เลือก Record ที่จะลบ
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_delete = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง DELETE สำหรับส่งไป MySQL
        $sql_delete = "delete from orders
                       // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
                       where md5(order_id) = '$val'";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_delete)){
            // header("Location: ...") = Redirect Browser ไป order.php?msg=delete
            header("Location: order.php?msg=delete");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order.php?msg=error
            header("Location: order.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // header("Location: ...") = Redirect Browser ไป order.php
    header("Location: order.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 20/22 : order_detail.php
หน้าที่: หน้า Admin จัดการรายการย่อยของคำสั่งซื้อ
เชื่อม: include conn.php; Form → order_detail_exc.php; Query เชื่อม orders/books
FLOW จำ: Admin guard → Search/Edit → JOIN books → Form/Table
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // $search = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้น
    $search = "";
    // $edit_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บข้อมูล Order Detail ที่โหลดมาแก้
    $edit_detail = NULL;

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["search"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["search"])){
        $search = $_GET["search"];
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["edit_id"] = ค่าจาก URL/Form method="get"
    if(isset($_GET["edit_id"])){
        // $edit_id = ตัวแปรที่เราสร้างเอง ใช้เก็บID Record ที่จะโหลดมาแก้
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $edit_id = intval($_GET["edit_id"]);

        // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
        // $sql_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
        $sql_edit = "select * from order_details where order_detail_id = $edit_id";
        // $result_edit = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        $result_edit = mysqli_query($conn,$sql_edit);
        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
        $edit_detail = mysqli_fetch_assoc($result_edit);
    }

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // INNER JOIN=เชื่อมเฉพาะแถวที่จับคู่กัน; ON=บอก Field ที่ใช้จับคู่
    // $sql_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_detail = "select order_details.*, books.book_title
                   from order_details
                   inner join books on order_details.book_id = books.book_id";

    // != = ไม่เท่ากัน; ตรงนี้เช็กว่าค่าด้านซ้ายไม่ใช่ข้อความว่าง
    if($search != ""){
        // $search_sql = ตัวแปรที่เราสร้างเอง ใช้เก็บคำค้นหลังจัดอักขระพิเศษ
        // mysqli_real_escape_string() = Function ของ MySQLi ใช้จัดอักขระพิเศษของข้อความก่อนนำไปประกอบ SQL
        $search_sql = mysqli_real_escape_string($conn,$search);

        // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
        $sql_detail .= " where order_details.order_detail_id like '%$search_sql%'
                         or order_details.order_id like '%$search_sql%'
                         or books.book_title like '%$search_sql%'";
    }

    // .= = ต่อข้อความใหม่ท้ายตัวแปรเดิม; ในงานใช้ต่อเงื่อนไข SQL ทีละส่วน
    $sql_detail .= " order by order_details.order_detail_id desc";
    // $result_detail = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_detail = mysqli_query($conn,$sql_detail);

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // $sql_order = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_order = "select * from orders order by order_id desc";
    // $result_order = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_order = mysqli_query($conn,$sql_order);

    // SELECT=อ่านข้อมูล, FROM=ระบุตาราง, WHERE=กรองเงื่อนไข
    // ORDER BY=เรียงผล; DESC=มากไปน้อย/ใหม่ไปเก่า; ASC=น้อยไปมาก
    // $sql_book = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง SELECT สำหรับส่งไป MySQL
    $sql_book = "select * from books order by book_title";
    // $result_book = ตัวแปรที่เราสร้างเอง ใช้เก็บผลจาก mysqli_query()
    // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
    $result_book = mysqli_query($conn,$sql_book);
?>

<!-- <!DOCTYPE html> = บอก Browser ว่าหน้านี้ใช้มาตรฐาน HTML5 -->
<!DOCTYPE html>
<html lang="th">

<head>
    <!-- UTF-8 = Encoding สำหรับภาษาไทย/Unicode -->
    <meta charset="UTF-8">

    <!-- viewport = ช่วยให้หน้า Responsive ตามขนาดมือถือ/หน้าจอ -->
    <meta name="viewport"
        content="width=device-width, initial-scale=1.0">

    <title>จัดการรายละเอียดคำสั่งซื้อ</title>

    <!-- Bootstrap 5.3.3 = CSS Framework สำเร็จรูป; class เช่น container,row,col,btn ใช้จัด Layout/Responsive -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

    <!-- Font Awesome = Library Icon; class fa-* เป็นชื่อ Icon สำเร็จรูป -->
    <link rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>

<body class="d-flex flex-column min-vh-100">

    <!-- Navbar Admin: แสดงชื่อจาก Session, Logout และเมนูไป book/category/user/order/order_detail -->
    <nav class="navbar bg-dark navbar-dark">
        <div class="container-fluid">

            <a href="book.php" class="navbar-brand">
                จัดการร้านหนังสือ
            </a>

            <span class="text-white ms-auto me-3">
                <?php echo $_SESSION["full_name"]; ?>
            </span>

            <a href="logout.php" class="btn btn-outline-light">
                ออกจากระบบ
            </a>

        </div>
    </nav>
    <div class="container mt-4 mb-5">
        <div class="mb-4">
            <a href="book.php" class="btn btn-outline-dark">หนังสือ</a>
            <a href="category.php" class="btn btn-outline-dark">หมวดหมู่</a>
            <a href="user.php" class="btn btn-outline-dark">ผู้ใช้</a>
            <a href="order.php" class="btn btn-outline-dark">คำสั่งซื้อ</a>
            <a href="order_detail.php" class="btn btn-dark">รายละเอียดคำสั่งซื้อ</a>
        </div>
        <h2 class="fw-bold mb-4">
            จัดการรายละเอียดคำสั่งซื้อ
        </h2>
        <?php
            // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["msg"] = ค่าจาก URL/Form method="get"
            if(isset($_GET["msg"])){
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "insert"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">เพิ่มรายละเอียดคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "update"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">แก้ไขรายละเอียดคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "delete"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-success">ลบรายละเอียดคำสั่งซื้อสำเร็จ</div>';
                }
                // == = เปรียบเทียบว่าเท่ากันหรือไม่
                if($_GET["msg"] == "error"){
                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                    echo '<div class="alert alert-danger">ไม่สามารถดำเนินการได้</div>';
                }
            }
        ?>
        <div class="border rounded p-3 mb-4">
            <?php
                if($edit_detail){
            ?>
                <h4 class="fw-bold mb-3">แก้ไขรายละเอียดคำสั่งซื้อ</h4>
                <!-- form ส่งข้อมูลไป order_detail_exc.php แบบ POST -->
                <form action="order_detail_exc.php" method="post">
                    <!-- name="chk" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["chk"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="update">
                    <!-- name="order_detail_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["order_detail_id"] -->
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="order_detail_id" value="<?php echo $edit_detail["order_detail_id"]; ?>">
            <?php
                // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                }else{
            ?>
                <h4 class="fw-bold mb-3">เพิ่มรายละเอียดคำสั่งซื้อ</h4>
                <!-- form ส่งข้อมูลไป order_detail_exc.php แบบ POST -->
                <form action="order_detail_exc.php" method="post">
                    <!-- type="hidden" = ไม่แสดงช่องบนหน้าแต่ยังส่ง name/value ไป Server; ไม่ใช่ระบบความปลอดภัย -->
                    <input type="hidden" name="chk" value="insert">
            <?php
                }
            ?>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">คำสั่งซื้อ</label>
                        <!-- name="order_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["order_id"] -->
                        <select name="order_id" class="form-select" required>
                            <?php
                                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                                while($order = mysqli_fetch_assoc($result_order)){
                                    // $selected = ตัวแปรที่เราสร้างเอง ใช้เก็บค่า selected สำหรับ option
                                    $selected = "";
                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_detail && $edit_detail["order_id"] == $order["order_id"]){
                                        $selected = "selected";
                                    }
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $order["order_id"] . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo 'คำสั่งซื้อ #' . $order["order_id"];
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">หนังสือ</label>
                        <!-- name="book_id" = Key ที่เราเป็นคนตั้งเอง; ฝั่ง PHP รับด้วย $_POST["book_id"] -->
                        <select name="book_id" class="form-select" required>
                            <?php
                                // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                                // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                                while($book = mysqli_fetch_assoc($result_book)){
                                    $selected = "";
                                    // == = เปรียบเทียบว่าเท่ากันหรือไม่
                                    if($edit_detail && $edit_detail["book_id"] == $book["book_id"]){
                                        $selected = "selected";
                                    }
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '<option value="' . $book["book_id"] . '" ' . $selected . '>';
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo $book["book_title"];
                                    // echo = คำของ PHP ใช้ส่งข้อความ/HTML ออกไปให้ Browser แสดง
                                    echo '</option>';
                                }
                            ?>
                        </select>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">จำนวน</label>
                        <input type="number"
                            name="quantity"
                            class="form-control"
                            min="1"
                            value="<?php if($edit_detail){ echo $edit_detail["quantity"]; } ?>"
                            required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label class="fw-bold">ราคาต่อหน่วย</label>
                        <input type="number"
                            name="unit_price"
                            class="form-control"
                            step="0.01"
                            min="0"
                            value="<?php if($edit_detail){ echo $edit_detail["unit_price"]; } ?>"
                            required>
                    </div>
                </div>
                <?php
                    if($edit_detail){
                ?>
                    <button type="submit" class="btn btn-success">
                        บันทึกการแก้ไข
                    </button>
                    <a href="order_detail.php" class="btn btn-secondary">
                        ยกเลิก
                    </a>
                <?php
                    // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
                    }else{
                ?>
                    <button type="submit" class="btn btn-success">
                        เพิ่มรายละเอียด
                    </button>
                <?php
                    }
                ?>
            </form>
        </div>
        <!-- form ส่งข้อมูลไป order_detail.php แบบ GET -->
        <form action="order_detail.php" method="get" class="d-flex mb-3">
            <input type="text"
                name="search"
                class="form-control me-2"
                placeholder="ค้นหา ID รายการ เลขคำสั่งซื้อ หรือชื่อหนังสือ"
                value="<?php echo $search; ?>">
            <button type="submit" class="btn btn-dark">
                <i class="fa-solid fa-magnifying-glass"></i>
                ค้นหา
            </button>
        </form>
        <div class="table-responsive">
            <table class="table table-bordered table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>คำสั่งซื้อ</th>
                        <th>หนังสือ</th>
                        <th>จำนวน</th>
                        <th>ราคาต่อหน่วย</th>
                        <th>รวม</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody>
                    <?php
                        // mysqli_fetch_assoc() = Function ของ MySQLi ใช้หยิบผล SELECT ออกมา 1 แถวเป็น Array ที่เรียกด้วยชื่อ Field
                        // while = Loop ทำซ้ำ; ในงานนี้มักใช้ mysqli_fetch_assoc() หยิบข้อมูลทีละแถวจนหมด
                        while($detail = mysqli_fetch_assoc($result_detail)){
                    ?>
                        <tr>
                            <td><?php echo $detail["order_detail_id"]; ?></td>
                            <td>#<?php echo $detail["order_id"]; ?></td>
                            <td><?php echo $detail["book_title"]; ?></td>
                            <td><?php echo $detail["quantity"]; ?></td>
                            <td><?php echo $detail["unit_price"]; ?> บาท</td>
                            <td><?php echo $detail["subtotal"]; ?> บาท</td>
                            <td>
                                <a href="order_detail.php?edit_id=<?php echo $detail["order_detail_id"]; ?>"
                                    class="btn btn-warning btn-sm">
                                    แก้ไข
                                </a>
                                <a href="order_detail_exc.php?val=<?php echo md5($detail["order_detail_id"]); ?>&chk=<?php echo md5("delete"); ?>"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('ต้องการลบรายละเอียดนี้หรือไม่?')">
                                    ลบ
                                </a>
                            </td>
                        </tr>
                    <?php
                        }
                    ?>
                </tbody>
            </table>
        </div>
    </div>
    <footer class="bg-dark text-white text-center py-2 mt-auto">
        <p class="mb-0">@2026 โลกเหนือหน้ากระดาษ</p>
    </footer>
</body>

</html>

==============================================================================================================
ไฟล์ 21/22 : order_detail_exc.php
หน้าที่: Processor ของ order_detail.php สำหรับ CRUD รายการย่อยและคำนวณ subtotal
เชื่อม: include conn.php; รับจาก order_detail.php; เสร็จ → order_detail.php?msg=...
FLOW จำ: Admin guard → quantity×unit_price → CRUD details → Redirect
==============================================================================================================
<?php
    // session_start() = Function ของ PHP ใช้เปิด Session เพื่อให้หน้าอ่าน/เขียน $_SESSION ได้
    session_start();
    // include = คำเฉพาะของ PHP ใช้เอา conn.php เข้ามา ทำให้ไฟล์นี้ได้ $conn ไป Query Database
    include "./connect/conn.php";

    // เช็กสิทธิ์ Admin: ถ้ายังไม่ Login หรือ role ไม่ใช่ admin ให้กลับ login.php
    // ! = ไม่, || = หรือ, != = ไม่เท่ากัน
    if(!isset($_SESSION["user_id"]) || $_SESSION["role"] != "admin"){
        // header("Location: ...") = Redirect Browser ไป login.php
        header("Location: login.php");
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "insert"){

        // $order_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสคำสั่งซื้อ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $order_id = intval($_POST["order_id"]);
        // $book_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสหนังสือ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_POST["book_id"]);
        // $quantity = ตัวแปรที่เราสร้างเอง ใช้เก็บจำนวน
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $quantity = intval($_POST["quantity"]);
        // $unit_price = ตัวแปรที่เราสร้างเอง ใช้เก็บราคาต่อหน่วย
        $unit_price = $_POST["unit_price"];

        // $subtotal = ตัวแปรที่เราสร้างเอง ใช้เก็บยอดย่อย
        $subtotal = $quantity * $unit_price;

        // INSERT INTO=เพิ่ม Record ใหม่; VALUES=ค่าที่จะบันทึก โดยลำดับต้องตรงกับ Field
        // $sql_insert = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง INSERT สำหรับส่งไป MySQL
        $sql_insert = "insert into order_details
        (order_id,book_id,quantity,unit_price,subtotal)
        values
        ($order_id,$book_id,$quantity,$unit_price,$subtotal)";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_insert)){
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=insert
            header("Location: order_detail.php?msg=insert");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=error
            header("Location: order_detail.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }

    // isset() = เช็กว่ามีค่านี้หรือยัง; $_POST["chk"] = ค่าจาก Form method="post"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    if(isset($_POST["chk"]) && $_POST["chk"] == "update"){

        // $order_detail_id = ตัวแปรที่เราสร้างเอง ใช้เก็บรหัสรายละเอียดคำสั่งซื้อ
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $order_detail_id = intval($_POST["order_detail_id"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $order_id = intval($_POST["order_id"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $book_id = intval($_POST["book_id"]);
        // intval() = Function ของ PHP ใช้แปลงค่าให้เป็นเลขจำนวนเต็ม
        $quantity = intval($_POST["quantity"]);
        $unit_price = $_POST["unit_price"];

        $subtotal = $quantity * $unit_price;

        // UPDATE=แก้ Record เดิม; SET=กำหนดค่าใหม่; WHERE=เลือก Record ที่จะแก้
        // $sql_update = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง UPDATE สำหรับส่งไป MySQL
        $sql_update = "update order_details set
        order_id = $order_id,
        book_id = $book_id,
        quantity = $quantity,
        unit_price = $unit_price,
        subtotal = $subtotal
        where order_detail_id = $order_detail_id";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_update)){
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=update
            header("Location: order_detail.php?msg=update");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=error
            header("Location: order_detail.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // isset() = เช็กว่ามีค่านี้หรือยัง; $_GET["chk"] = ค่าจาก URL/Form method="get"
    // && = และ หมายถึงเงื่อนไขทั้งสองด้านต้องจริง
    // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
    if(isset($_GET["chk"]) && $_GET["chk"] == md5("delete")){
        // $val = ตัวแปรที่เราสร้างเอง ใช้เก็บค่าที่ส่งมาใช้ระบุ Record ตอนลบ
        $val = $_GET["val"];
        // DELETE FROM=ลบ Record; WHERE=เลือก Record ที่จะลบ
        // ' ' = Single Quote ใน SQL ใช้ครอบค่าข้อความ เช่น '$email'
        // $sql_delete = ตัวแปรที่เราสร้างเอง ใช้เก็บคำสั่ง DELETE สำหรับส่งไป MySQL
        $sql_delete = "delete from order_details
                       // md5() = Function สร้าง Hash; งานนี้ใช้ซ่อนค่า ID/คำ delete ใน URL (ไม่ใช่การเข้ารหัสแบบถอดกลับ)
                       where md5(order_detail_id) = '$val'";
        // mysqli_query() = Function ของ MySQLi ใช้ส่ง SQL ไป MySQL ผ่าน $conn
        if(mysqli_query($conn,$sql_delete)){
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=delete
            header("Location: order_detail.php?msg=delete");
        // else = ถ้า if ก่อนหน้าไม่จริง ให้ทำส่วนนี้แทน
        }else{
            // header("Location: ...") = Redirect Browser ไป order_detail.php?msg=error
            header("Location: order_detail.php?msg=error");
        }
        // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
        exit();
    }
    // header("Location: ...") = Redirect Browser ไป order_detail.php
    header("Location: order_detail.php");
    // exit() = หยุด Script ทันที มักใช้หลัง Redirect เพื่อไม่ให้โค้ดด้านล่างทำงานต่อ
    exit();
?>

==============================================================================================================
ไฟล์ 22/22 : bookstore.sql
หน้าที่: SQL Dump สำหรับสร้าง Database Structure, ข้อมูลตัวอย่าง, Key และ Foreign Key
เชื่อม: Import เข้า MySQL/phpMyAdmin; PHP ทุกหน้าจะ Query ตารางที่ไฟล์นี้สร้าง
FLOW จำ: CREATE TABLE → INSERT data → Key/AUTO_INCREMENT → Foreign Key → COMMIT
==============================================================================================================
-- phpMyAdmin SQL Dump
-- version 5.2.1
-- https://www.phpmyadmin.net/
--
-- Host: 127.0.0.1
-- Generation Time: Aug 16, 2026 at 12:17 PM
-- Server version: 10.4.32-MariaDB
-- PHP Version: 8.2.12

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Database: `bookstore`
--

-- --------------------------------------------------------

--
-- Table structure for table `books`
--

-- CREATE TABLE = สร้างตารางและกำหนด Field/ชนิดข้อมูล
CREATE TABLE `books` (
  `book_id` int(10) UNSIGNED NOT NULL,
  `category_id` int(10) UNSIGNED NOT NULL,
  `book_title` varchar(255) NOT NULL,
  `author_name` varchar(150) NOT NULL,
  `price` decimal(10,2) UNSIGNED NOT NULL,
  `stock` int(10) UNSIGNED NOT NULL DEFAULT 0,
  `description` text DEFAULT NULL,
  `cover_image` varchar(255) DEFAULT NULL,
  `release_date` date DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp()
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `books`
--

-- INSERT INTO = เพิ่มข้อมูลตัวอย่างลงตาราง
INSERT INTO `books` (`book_id`, `category_id`, `book_title`, `author_name`, `price`, `stock`, `description`, `cover_image`, `release_date`, `created_at`) VALUES
(1, 6, 'เปิดโลกวิทยาศาสตร์', 'ดร.กิตติพงษ์ แสงดาว', 259.00, 20, 'หนังสือวิทยาศาสตร์พื้นฐานที่พาผู้อ่านไปรู้จักเรื่องราวรอบตัวตั้งแต่สิ่งมีชีวิต พลังงาน สสาร ไปจนถึงปรากฏการณ์ทางธรรมชาติ เนื้อหาอธิบายด้วยภาษาที่เข้าใจง่ายและยกตัวอย่างจากชีวิตประจำวัน เหมาะสำหรับผู้ที่ต้องการเริ่มเรียนรู้วิทยาศาสตร์โดยไม่จำเป็นต้องมีพื้นฐานมาก่อน', 'open_science.png', '2026-01-10', '2026-08-03 07:37:22'),
(2, 7, 'ประวัติศาสตร์โลกฉบับเข้าใจง่าย', 'วรชัย ศรีสมัย', 289.00, 15, 'พาผู้อ่านสำรวจประวัติศาสตร์โลกตั้งแต่อารยธรรมโบราณ การกำเนิดเมืองและจักรวรรดิ ไปจนถึงเหตุการณ์สำคัญที่มีผลต่อโลกในยุคปัจจุบัน เนื้อหาเรียบเรียงตามลำดับเวลาและอธิบายความเชื่อมโยงของแต่ละเหตุการณ์อย่างเข้าใจง่าย เหมาะสำหรับผู้ที่อยากเห็นภาพรวมของประวัติศาสตร์โลกโดยไม่รู้สึกว่าเนื้อหาหนักเกินไป', 'world_history.png', '2026-01-15', '2026-08-03 07:37:22'),
(3, 8, 'การเงินพื้นฐานวัยเริ่มต้น', 'ธนกร มั่งคั่ง', 239.00, 25, 'หนังสือสำหรับผู้เริ่มต้นที่ต้องการเข้าใจเรื่องการเงินส่วนบุคคล ตั้งแต่การจัดสรรรายรับ การควบคุมรายจ่าย การออมเงิน ไปจนถึงแนวคิดพื้นฐานเกี่ยวกับการลงทุน เนื้อหาช่วยให้ผู้อ่านวางแผนการเงินอย่างเป็นขั้นตอนและเห็นความสำคัญของการสร้างวินัยทางการเงิน เหมาะสำหรับนักเรียน นักศึกษา และผู้ที่เพิ่งเริ่มจัดการเงินด้วยตนเอง', 'basic_finance.png', '2026-01-20', '2026-08-03 07:37:22'),
(4, 2, 'ผจญภัยในป่ามหัศจรรย์', 'พิมพ์ชนก สดใส', 159.00, 30, 'เรื่องราวการผจญภัยของเด็กกลุ่มหนึ่งที่หลงเข้าไปในป่ามหัศจรรย์ซึ่งเต็มไปด้วยสัตว์พูดได้ พืชประหลาด และเส้นทางลึกลับ ระหว่างการหาทางกลับบ้าน พวกเขาต้องช่วยเหลือกันและเรียนรู้ถึงความสำคัญของมิตรภาพ ความกล้าหาญ และการไม่ทิ้งเพื่อน เหมาะสำหรับเด็กและครอบครัวที่ชื่นชอบเรื่องราวสนุกสนานและจินตนาการ', 'magic_forest.png', '2026-02-01', '2026-08-03 07:37:22'),
(5, 2, 'เมืองเมฆป่วนของแก๊งตัวจิ๋ว', 'นันทิชา จินตนา', 169.00, 28, 'กลุ่มตัวละครตัวจิ๋วออกเดินทางสำรวจเมืองที่ตั้งอยู่บนก้อนเมฆ แต่ความสงบของเมืองกลับถูกแทนที่ด้วยเหตุการณ์วุ่นวายที่ไม่มีใครคาดคิด พวกเขาต้องร่วมมือกันแก้ปัญหาและค้นหาสาเหตุของความผิดปกติที่เกิดขึ้น เรื่องราวเต็มไปด้วยมุกสนุก การผจญภัย และข้อคิดเรื่องการทำงานร่วมกัน', 'cloud_city.png', '2026-02-05', '2026-08-03 07:37:22'),
(6, 3, 'นักดาบเงาจันทร์', 'คุโรฮะ', 199.00, 18, 'นักดาบหนุ่มผู้ใช้ชีวิตอยู่ภายใต้เงาของอดีตต้องออกเดินทางเพื่อตามหาความจริงเกี่ยวกับวิชาดาบต้องห้าม ทุกค่ำคืนที่แสงจันทร์ปรากฏ ศัตรูจากอดีตก็เริ่มเผยตัวและนำเขาเข้าใกล้ความจริงมากขึ้น การต่อสู้แต่ละครั้งไม่ได้ทดสอบเพียงฝีมือดาบ แต่ยังทดสอบความเชื่อและการตัดสินใจของเขาด้วย', 'moon_swordsman.png', '2026-02-10', '2026-08-03 07:37:22'),
(7, 3, 'ชมรมลับหลังเลิกเรียน', 'ปั๊ปปี้เบอร์รี', 189.00, 22, 'ชมรมธรรมดาหลังเลิกเรียนกลับซ่อนความลับบางอย่างที่นักเรียนทั่วไปไม่เคยรู้ สมาชิกแต่ละคนต่างมีเหตุผลของตนเองในการเข้าร่วม และความสัมพันธ์ของพวกเขาก็ค่อย ๆ เปลี่ยนไปจากคนแปลกหน้าเป็นเพื่อนสนิท เรื่องราวผสมทั้งชีวิตในโรงเรียน ความรัก ความลับ และเหตุการณ์วุ่นวายที่เกิดขึ้นเมื่อความจริงของชมรมเริ่มถูกเปิดเผย', 'secret_club.png', '2026-02-15', '2026-08-03 07:37:22'),
(8, 4, 'หอคอยแห่งดวงดาว', 'ฮันซอจุน', 249.00, 17, 'ในโลกที่มีหอคอยลึกลับปรากฏขึ้น ผู้ที่ได้รับเลือกเท่านั้นจึงจะสามารถเข้าสู่หอคอยแห่งดวงดาวได้ ตัวเอกต้องผ่านบททดสอบในแต่ละชั้นและเผชิญหน้ากับทั้งมิตรและศัตรูที่มีเป้าหมายแตกต่างกัน ยิ่งขึ้นไปสูงเท่าไร ความจริงเกี่ยวกับหอคอยและเหตุผลที่เขาถูกเลือกก็ยิ่งชัดเจนขึ้น', 'star_tower.png', '2026-02-20', '2026-08-03 07:37:22'),
(9, 4, 'สัญญาใต้แสงจันทร์', 'คิมยูนา', 259.00, 16, 'หญิงสาวผู้เคยให้คำสัญญาในคืนพระจันทร์เต็มดวงได้กลับมาพบกับขุนนางลึกลับที่ดูเหมือนจะรู้เรื่องราวในอดีตของเธอมากกว่าที่ควรจะเป็น ความสัมพันธ์ของทั้งสองค่อย ๆ พัฒนาไปพร้อมกับความลับของตระกูลและพลังบางอย่างที่ถูกปิดบัง เรื่องราวผสมความโรแมนติก แฟนตาซี และปริศนาที่เกี่ยวข้องกับคำสัญญาในอดีต', 'moonlight_promise.png', '2026-02-25', '2026-08-03 07:37:22'),
(10, 4, 'จอมเวทคืนชีพ', 'พัคจีฮุน', 269.00, 19, 'จอมเวทผู้เคยยืนอยู่บนจุดสูงสุดของโลกกลับมามีชีวิตอีกครั้งในยุคที่ทุกอย่างเปลี่ยนไป เขาพบว่าหลายสิ่งที่เคยรู้จักได้สูญหาย และเหตุการณ์บางอย่างในอดีตอาจไม่ได้เป็นอย่างที่เขาเคยเชื่อ การเริ่มต้นชีวิตใหม่ครั้งนี้จึงไม่ใช่เพียงการกลับมาสู่จุดสูงสุด แต่เป็นโอกาสในการเปลี่ยนแปลงชะตากรรมของตนเองและผู้คนรอบตัว', 'reborn_mage.png', '2026-03-01', '2026-08-03 07:37:22'),
(11, 10, 'ครัวง่ายสไตล์บ้าน', 'เชฟมะลิ', 229.00, 24, 'รวมสูตรอาหารสำหรับทำกินเองที่บ้านโดยเน้นขั้นตอนง่าย วัตถุดิบหาได้ทั่วไป และไม่จำเป็นต้องมีอุปกรณ์ทำครัวซับซ้อน มีทั้งเมนูอาหารจานเดียว เมนูกับข้าว และเมนูสำหรับทำร่วมกับครอบครัว แต่ละสูตรอธิบายขั้นตอนอย่างเป็นลำดับ เหมาะสำหรับนักทำอาหารมือใหม่ที่ต้องการเริ่มเข้าครัวด้วยตนเอง', 'easy_home_cooking.png', '2026-03-05', '2026-08-03 07:37:22'),
(12, 6, 'จักรวาลใกล้ตัว', 'ดร.ธีรภัทร อวกาศ', 279.00, 14, 'พาผู้อ่านออกเดินทางจากโลกไปสำรวจระบบสุริยะ ดาวฤกษ์ เนบิวลา และกาแล็กซีที่อยู่ไกลออกไป เนื้อหาอธิบายปรากฏการณ์ทางดาราศาสตร์ด้วยภาษาที่เข้าใจง่ายพร้อมเชื่อมโยงกับสิ่งที่เราสามารถสังเกตเห็นจากโลก ช่วยให้ผู้อ่านเข้าใจว่าจักรวาลมีขนาดใหญ่เพียงใดและโลกของเราเป็นส่วนหนึ่งของจักรวาลอย่างไร', 'near_universe.png', '2026-03-10', '2026-08-03 07:37:22'),
(13, 9, 'นิสัยเล็ก เปลี่ยนชีวิตใหญ่', 'ศิริพร พัฒนาตน', 219.00, 26, 'แนวทางพัฒนาตนเองด้วยการเริ่มเปลี่ยนพฤติกรรมเล็ก ๆ ที่สามารถทำได้จริงในชีวิตประจำวัน หนังสืออธิบายว่าการสร้างนิสัยอย่างต่อเนื่องสามารถส่งผลต่อการเรียน การทำงาน สุขภาพ และเป้าหมายในระยะยาวได้อย่างไร เหมาะสำหรับผู้ที่เคยตั้งเป้าหมายไว้หลายครั้งแต่ยังไม่สามารถรักษาวินัยให้ต่อเนื่องได้', 'small_habits.png', '2026-03-15', '2026-08-03 07:37:22'),
(14, 2, 'กัปตันน้อยตะลุยทะเลสีรุ้ง', 'น้องฟ้าใส', 149.00, 32, 'กัปตันตัวน้อยออกเดินเรือไปยังทะเลสีรุ้งเพื่อตามหาเกาะในตำนานที่ไม่มีอยู่ในแผนที่ ระหว่างทางเขาและเพื่อน ๆ ต้องพบกับสัตว์ทะเลแปลกประหลาด พายุ และปริศนาที่ต้องช่วยกันแก้ เรื่องราวเน้นความสนุก จินตนาการ ความกล้าหาญ และการช่วยเหลือกัน เหมาะสำหรับเด็กที่ชื่นชอบการผจญภัย', 'rainbow_sea.png', '2026-03-20', '2026-08-03 07:37:22'),
(15, 2, 'โรงเรียนไดโนป่วน', 'พลอยชมพู', 159.00, 29, 'โรงเรียนแห่งหนึ่งไม่ได้มีนักเรียนธรรมดา แต่เต็มไปด้วยไดโนเสาร์ตัวน้อยหลากหลายสายพันธุ์ที่มีนิสัยแตกต่างกัน ในแต่ละวันพวกเขาต้องเรียน เล่น และแก้ปัญหาความวุ่นวายที่เกิดขึ้นในโรงเรียน เรื่องราวเต็มไปด้วยความสนุกและมิตรภาพ พร้อมสอดแทรกเรื่องการแบ่งปัน การรับผิดชอบ และการอยู่ร่วมกับผู้อื่น', 'dino_school.png', '2026-03-25', '2026-08-03 07:37:22'),
(16, 3, 'นักสืบเงาไซเบอร์', 'เรย์ คุโรซากิ', 209.00, 18, 'ในมหานครอนาคตที่เทคโนโลยีเชื่อมโยงทุกชีวิต นักสืบหนุ่มได้รับคดีเกี่ยวกับบุคคลที่หายตัวไปอย่างไร้ร่องรอย การสืบสวนพาเขาเข้าสู่โลกใต้ดินของข้อมูล ปัญญาประดิษฐ์ และองค์กรลึกลับที่สามารถควบคุมข้อมูลของประชาชนได้ ยิ่งเข้าใกล้ความจริง เขาก็ยิ่งพบว่าคดีนี้อาจเกี่ยวข้องกับอดีตของตัวเอง', 'cyber_detective.png', '2026-04-01', '2026-08-03 07:37:22'),
(17, 3, 'โรงเรียนหมัดมังกร', 'ทาเคชิ ริว', 199.00, 20, 'โรงเรียนแห่งนี้มีชื่อเสียงด้านศิลปะการต่อสู้และรวบรวมนักสู้ฝีมือดีจากทั่วประเทศ ตัวเอกซึ่งเข้ามาในฐานะนักเรียนใหม่ต้องพิสูจน์ตัวเองผ่านการแข่งขันและการฝึกฝนที่เข้มข้น ระหว่างทางเขาได้พบทั้งคู่แข่ง เพื่อน และอาจารย์ที่ช่วยให้เขาเข้าใจว่าการเป็นนักสู้ที่แข็งแกร่งไม่ได้ขึ้นอยู่กับพลังเพียงอย่างเดียว', 'dragon_fist_school.png', '2026-04-05', '2026-08-03 07:37:22'),
(18, 4, 'ท่านหญิงผู้เขียนชะตาใหม่', 'อียอนฮวา', 269.00, 15, 'หญิงสาวผู้เคยใช้ชีวิตตามเส้นทางที่ผู้อื่นกำหนดได้รับโอกาสย้อนกลับมาเริ่มต้นชีวิตของตนเองอีกครั้ง คราวนี้เธอตัดสินใจไม่ยอมให้เหตุการณ์เดิมเกิดขึ้นซ้ำและเริ่มเปลี่ยนแปลงความสัมพันธ์กับผู้คนรอบตัว การตัดสินใจเล็ก ๆ ของเธอกลับส่งผลต่ออนาคตมากกว่าที่คาดไว้ พร้อมนำไปสู่ทั้งความรัก การเมืองในตระกูล และความลับที่ไม่เคยได้รับรู้ในชีวิตก่อน', 'lady_new_destiny.png', '2026-04-10', '2026-08-03 07:37:22'),
(19, 4, 'ฮันเตอร์แห่งหอคอยมืด', 'ชเวมินซอก', 279.00, 13, 'เมื่อหอคอยมืดปรากฏขึ้นและปล่อยสัตว์ประหลาดออกมาคุกคามโลก เหล่าฮันเตอร์จึงถูกส่งเข้าไปสำรวจและหยุดยั้งภัยพิบัติ ตัวเอกซึ่งเคยเป็นฮันเตอร์ธรรมดากลับพบความลับบางอย่างภายในหอคอยที่เชื่อมโยงกับตัวเขาเอง การต่อสู้เพื่อเอาชีวิตรอดจึงกลายเป็นการค้นหาความจริงเกี่ยวกับต้นกำเนิดของหอคอยและภัยที่กำลังเข้าใกล้โลก', 'dark_tower_hunter.png', '2026-04-15', '2026-08-03 07:37:22'),
(20, 4, 'รุ่นพี่คนนี้มีใจหรือเปล่า', 'ฮันซูอา', 239.00, 21, 'ชีวิตมหาวิทยาลัยของรุ่นน้องคนหนึ่งเริ่มวุ่นวายขึ้นเมื่อเธอต้องพบกับรุ่นพี่จอมแกล้งอยู่บ่อยครั้ง แม้ภายนอกทั้งสองจะชอบเถียงและแกล้งกัน แต่เหตุการณ์หลายอย่างกลับทำให้พวกเขาต้องใช้เวลาร่วมกันมากขึ้น ความสัมพันธ์ที่เริ่มต้นจากความวุ่นวายจึงค่อย ๆ เปลี่ยนเป็นความรู้สึกที่ทั้งคู่ยังไม่แน่ใจว่าจะเรียกว่าอะไร', 'senior_love.png', '2026-04-20', '2026-08-03 07:37:22');

-- --------------------------------------------------------

--
-- Table structure for table `categories`
--

-- CREATE TABLE = สร้างตารางและกำหนด Field/ชนิดข้อมูล
CREATE TABLE `categories` (
  `category_id` int(10) UNSIGNED NOT NULL,
  `parent_id` int(10) UNSIGNED DEFAULT NULL COMMENT 'รหัสประเภทหลัก',
  `category_name` varchar(100) NOT NULL,
  `description` text DEFAULT NULL,
  `category_image` varchar(255) DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp()
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `categories`
--

-- INSERT INTO = เพิ่มข้อมูลตัวอย่างลงตาราง
INSERT INTO `categories` (`category_id`, `parent_id`, `category_name`, `description`, `category_image`, `created_at`) VALUES
(1, NULL, 'หนังสือความรู้', 'หนังสือสำหรับเพิ่มความรู้ในด้านต่าง ๆ', 'knowledge.jpg', '2026-08-03 07:17:31'),
(2, 13, 'การ์ตูนเด็ก', 'หนังสือการ์ตูนสำหรับเด็กและครอบครัว', 'children_cartoon.jpg', '2026-08-03 07:17:31'),
(3, 11, 'มังงะ', 'หนังสือการ์ตูนสไตล์ญี่ปุ่น', 'manga.jpg', '2026-08-03 07:17:31'),
(4, 11, 'มังฮวา', 'หนังสือการ์ตูนสไตล์เกาหลี', 'manhwa.jpg', '2026-08-03 07:17:31'),
(5, 12, 'นิยายแฟนตาซี', 'นิยายเกี่ยวกับเวทมนตร์และโลกจินตนาการ', 'fantasy.jpg', '2026-08-03 07:17:31'),
(6, 1, 'วิทยาศาสตร์', 'หนังสือเกี่ยวกับวิทยาศาสตร์และธรรมชาติ', 'science.jpg', '2026-08-03 07:17:31'),
(7, 1, 'ประวัติศาสตร์', 'หนังสือเกี่ยวกับเหตุการณ์และเรื่องราวในอดีต', 'history.jpg', '2026-08-03 07:17:31'),
(8, 1, 'การเงินและการลงทุน', 'หนังสือเกี่ยวกับการเงิน การออม และการลงทุน', 'finance.jpg', '2026-08-03 07:17:31'),
(9, 1, 'พัฒนาตนเอง', 'หนังสือสำหรับพัฒนาความคิดและทักษะชีวิต', 'self_development.jpg', '2026-08-03 07:17:31'),
(10, 1, 'อาหารและการทำครัว', 'หนังสือสูตรอาหารและความรู้เกี่ยวกับการทำอาหาร', 'cooking.jpg', '2026-08-03 07:17:31'),
(11, NULL, 'มังงะ / มังฮวา', 'หนังสือการ์ตูนสไตล์ญี่ปุ่นและเกาหลี', NULL, '2026-08-06 12:18:51'),
(12, NULL, 'นิยาย', 'หนังสือนิยายและวรรณกรรมประเภทต่าง ๆ', NULL, '2026-08-06 12:18:51'),
(13, NULL, 'หนังสือเด็ก', 'หนังสือสำหรับเด็ก เสริมจินตนาการ ความรู้ และความสนุก', NULL, '2026-08-06 12:26:55');

-- --------------------------------------------------------

--
-- Table structure for table `orders`
--

-- CREATE TABLE = สร้างตารางและกำหนด Field/ชนิดข้อมูล
CREATE TABLE `orders` (
  `order_id` int(10) UNSIGNED NOT NULL,
  `user_id` int(10) UNSIGNED NOT NULL,
  `total_amount` decimal(10,2) UNSIGNED NOT NULL DEFAULT 0.00,
  `shipping_address` text NOT NULL,
  `payment_method` enum('โอนเงิน','เก็บเงินปลายทาง') NOT NULL DEFAULT 'โอนเงิน',
  `payment_slip` varchar(255) DEFAULT NULL,
  `order_status` enum('รอชำระเงิน','รอตรวจสอบ','กำลังเตรียมสินค้า','จัดส่งแล้ว','เสร็จสิ้น','ยกเลิก') NOT NULL DEFAULT 'รอชำระเงิน',
  `created_at` timestamp NOT NULL DEFAULT current_timestamp()
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `orders`
--

-- INSERT INTO = เพิ่มข้อมูลตัวอย่างลงตาราง
INSERT INTO `orders` (`order_id`, `user_id`, `total_amount`, `shipping_address`, `payment_method`, `payment_slip`, `order_status`, `created_at`) VALUES
(1, 2, 418.00, '123 ตำบลหน้าเมือง อำเภอเมือง จังหวัดราชบุรี 70000', 'โอนเงิน', 'slip_order_001.jpg', 'เสร็จสิ้น', '2026-07-20 03:15:00'),
(2, 3, 398.00, '45 ตำบลดอนตะโก อำเภอเมือง จังหวัดราชบุรี 70000', 'เก็บเงินปลายทาง', NULL, 'จัดส่งแล้ว', '2026-07-22 06:30:00'),
(3, 4, 508.00, '89 ตำบลบ้านโป่ง อำเภอบ้านโป่ง จังหวัดราชบุรี 70110', 'โอนเงิน', 'slip_order_003.jpg', 'กำลังเตรียมสินค้า', '2026-07-24 02:45:00'),
(4, 5, 448.00, '67 ตำบลสวนผึ้ง อำเภอสวนผึ้ง จังหวัดราชบุรี 70180', 'โอนเงิน', 'slip_order_004.jpg', 'รอตรวจสอบ', '2026-07-25 09:20:00'),
(5, 6, 457.00, '31 ตำบลโพธาราม อำเภอโพธาราม จังหวัดราชบุรี 70120', 'เก็บเงินปลายทาง', NULL, 'กำลังเตรียมสินค้า', '2026-07-27 04:05:00'),
(6, 7, 408.00, '52 ตำบลเจดีย์หัก อำเภอเมือง จังหวัดราชบุรี 70000', 'โอนเงิน', 'slip_order_006.jpg', 'จัดส่งแล้ว', '2026-07-28 07:40:00'),
(7, 8, 508.00, '74 ตำบลดำเนินสะดวก อำเภอดำเนินสะดวก จังหวัดราชบุรี 70130', 'เก็บเงินปลายทาง', NULL, 'รอชำระเงิน', '2026-07-30 01:55:00'),
(8, 9, 568.00, '96 ตำบลบางแพ อำเภอบางแพ จังหวัดราชบุรี 70160', 'โอนเงิน', 'slip_order_008.jpg', 'รอตรวจสอบ', '2026-07-31 10:10:00'),
(9, 10, 478.00, '18 ตำบลจอมบึง อำเภอจอมบึง จังหวัดราชบุรี 70150', 'เก็บเงินปลายทาง', NULL, 'ยกเลิก', '2026-08-01 05:25:00'),
(10, 2, 717.00, '123 ตำบลหน้าเมือง อำเภอเมือง จังหวัดราชบุรี 70000', 'โอนเงิน', 'slip_order_010.jpg', 'รอชำระเงิน', '2026-08-03 02:30:00');

-- --------------------------------------------------------

--
-- Table structure for table `order_details`
--

-- CREATE TABLE = สร้างตารางและกำหนด Field/ชนิดข้อมูล
CREATE TABLE `order_details` (
  `order_detail_id` int(10) UNSIGNED NOT NULL,
  `order_id` int(10) UNSIGNED NOT NULL,
  `book_id` int(10) UNSIGNED NOT NULL,
  `quantity` int(10) UNSIGNED NOT NULL DEFAULT 1,
  `unit_price` decimal(10,2) UNSIGNED NOT NULL,
  `subtotal` decimal(10,2) UNSIGNED NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `order_details`
--

-- INSERT INTO = เพิ่มข้อมูลตัวอย่างลงตาราง
INSERT INTO `order_details` (`order_detail_id`, `order_id`, `book_id`, `quantity`, `unit_price`, `subtotal`) VALUES
(1, 1, 1, 1, 259.00, 259.00),
(2, 1, 4, 1, 159.00, 159.00),
(3, 2, 6, 1, 199.00, 199.00),
(4, 2, 17, 1, 199.00, 199.00),
(5, 3, 10, 1, 269.00, 269.00),
(6, 3, 3, 1, 239.00, 239.00),
(7, 4, 1, 1, 259.00, 259.00),
(8, 4, 7, 1, 189.00, 189.00),
(9, 5, 14, 2, 149.00, 298.00),
(10, 5, 15, 1, 159.00, 159.00),
(11, 6, 16, 1, 209.00, 209.00),
(12, 6, 17, 1, 199.00, 199.00),
(13, 7, 18, 1, 269.00, 269.00),
(14, 7, 20, 1, 239.00, 239.00),
(15, 8, 2, 1, 289.00, 289.00),
(16, 8, 12, 1, 279.00, 279.00),
(17, 9, 3, 1, 239.00, 239.00),
(18, 9, 20, 1, 239.00, 239.00),
(19, 10, 15, 1, 159.00, 159.00),
(20, 10, 12, 1, 279.00, 279.00),
(21, 10, 19, 1, 279.00, 279.00);

-- --------------------------------------------------------

--
-- Table structure for table `users`
--

-- CREATE TABLE = สร้างตารางและกำหนด Field/ชนิดข้อมูล
CREATE TABLE `users` (
  `user_id` int(10) UNSIGNED NOT NULL,
  `full_name` varchar(150) NOT NULL,
  `email` varchar(150) NOT NULL,
  `password` varchar(20) NOT NULL,
  `phone` varchar(20) DEFAULT NULL,
  `address` text DEFAULT NULL,
  `role` enum('member','admin') NOT NULL DEFAULT 'member',
  `created_at` timestamp NOT NULL DEFAULT current_timestamp()
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `users`
--

-- INSERT INTO = เพิ่มข้อมูลตัวอย่างลงตาราง
INSERT INTO `users` (`user_id`, `full_name`, `email`, `password`, `phone`, `address`, `role`, `created_at`) VALUES
(1, 'ผู้ดูแลระบบ', 'admin@bookworld.com', '123456', '0811111111', 'ร้านโลกเหนือหน้ากระดาษ จังหวัดราชบุรี', 'admin', '2026-08-03 07:23:55'),
(2, 'สมชาย ใจดี', 'somchai@example.com', '123456', '0822222222', '123 ตำบลหน้าเมือง อำเภอเมือง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(3, 'สมหญิง รักอ่าน', 'somying@example.com', '123456', '0833333333', '45 ตำบลดอนตะโก อำเภอเมือง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(4, 'กิตติศักดิ์ แสงทอง', 'kittisak@example.com', '123456', '0844444444', '89 ตำบลบ้านโป่ง อำเภอบ้านโป่ง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(5, 'พรทิพย์ จันทร์ดี', 'porntip@example.com', '123456', '0855555555', '67 ตำบลสวนผึ้ง อำเภอสวนผึ้ง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(6, 'ธนกร นักอ่าน', 'thanakorn@example.com', '123456', '0866666666', '31 ตำบลโพธาราม อำเภอโพธาราม จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(7, 'ชลธิชา รักหนังสือ', 'chonticha@example.com', '123456', '0877777777', '52 ตำบลเจดีย์หัก อำเภอเมือง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(8, 'ณัฐวุฒิ ผู้อ่าน', 'nattawut@example.com', '123456', '0888888888', '74 ตำบลดำเนินสะดวก อำเภอดำเนินสะดวก จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(9, 'ศิริพร อ่านเพลิน', 'siriporn@example.com', '123456', '0899999999', '96 ตำบลบางแพ อำเภอบางแพ จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55'),
(10, 'วรชัย หนังสือดี', 'worachai@example.com', '123456', '0800000000', '18 ตำบลจอมบึง อำเภอจอมบึง จังหวัดราชบุรี', 'member', '2026-08-03 07:23:55');

--
-- Indexes for dumped tables
--

--
-- Indexes for table `books`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `books`
-- PRIMARY KEY = รหัสหลักที่ระบุ Record ไม่ให้ซ้ำ
  ADD PRIMARY KEY (`book_id`),
  ADD KEY `category_id` (`category_id`);

--
-- Indexes for table `categories`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `categories`
-- PRIMARY KEY = รหัสหลักที่ระบุ Record ไม่ให้ซ้ำ
  ADD PRIMARY KEY (`category_id`),
  ADD UNIQUE KEY `category_name` (`category_name`),
  ADD KEY `parent_id` (`parent_id`);

--
-- Indexes for table `orders`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `orders`
-- PRIMARY KEY = รหัสหลักที่ระบุ Record ไม่ให้ซ้ำ
  ADD PRIMARY KEY (`order_id`),
  ADD KEY `user_id` (`user_id`);

--
-- Indexes for table `order_details`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `order_details`
-- PRIMARY KEY = รหัสหลักที่ระบุ Record ไม่ให้ซ้ำ
  ADD PRIMARY KEY (`order_detail_id`),
  ADD KEY `order_id` (`order_id`),
  ADD KEY `book_id` (`book_id`);

--
-- Indexes for table `users`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `users`
-- PRIMARY KEY = รหัสหลักที่ระบุ Record ไม่ให้ซ้ำ
  ADD PRIMARY KEY (`user_id`),
  ADD UNIQUE KEY `email` (`email`);

--
-- AUTO_INCREMENT for dumped tables
--

--
-- AUTO_INCREMENT for table `books`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `books`
-- AUTO_INCREMENT = ให้ MySQL สร้างเลข ID ถัดไปอัตโนมัติ
  MODIFY `book_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=21;

--
-- AUTO_INCREMENT for table `categories`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `categories`
-- AUTO_INCREMENT = ให้ MySQL สร้างเลข ID ถัดไปอัตโนมัติ
  MODIFY `category_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=14;

--
-- AUTO_INCREMENT for table `orders`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `orders`
-- AUTO_INCREMENT = ให้ MySQL สร้างเลข ID ถัดไปอัตโนมัติ
  MODIFY `order_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=11;

--
-- AUTO_INCREMENT for table `order_details`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `order_details`
-- AUTO_INCREMENT = ให้ MySQL สร้างเลข ID ถัดไปอัตโนมัติ
  MODIFY `order_detail_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=22;

--
-- AUTO_INCREMENT for table `users`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `users`
-- AUTO_INCREMENT = ให้ MySQL สร้างเลข ID ถัดไปอัตโนมัติ
  MODIFY `user_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=11;

--
-- Constraints for dumped tables
--

--
-- Constraints for table `books`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `books`
-- FOREIGN KEY = เชื่อม Field นี้ไปยัง Primary Key ของอีกตาราง
  ADD CONSTRAINT `fk_books_categories` FOREIGN KEY (`category_id`) REFERENCES `categories` (`category_id`) ON UPDATE CASCADE;

--
-- Constraints for table `categories`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `categories`
-- FOREIGN KEY = เชื่อม Field นี้ไปยัง Primary Key ของอีกตาราง
  ADD CONSTRAINT `fk_categories_parent` FOREIGN KEY (`parent_id`) REFERENCES `categories` (`category_id`) ON UPDATE CASCADE;

--
-- Constraints for table `orders`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `orders`
-- FOREIGN KEY = เชื่อม Field นี้ไปยัง Primary Key ของอีกตาราง
  ADD CONSTRAINT `fk_orders_users` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`) ON UPDATE CASCADE;

--
-- Constraints for table `order_details`
--
-- ALTER TABLE = แก้โครงสร้างตาราง เช่นเพิ่ม Key/AUTO_INCREMENT/Foreign Key
ALTER TABLE `order_details`
-- FOREIGN KEY = เชื่อม Field นี้ไปยัง Primary Key ของอีกตาราง
  ADD CONSTRAINT `fk_order_details_books` FOREIGN KEY (`book_id`) REFERENCES `books` (`book_id`) ON UPDATE CASCADE,
-- FOREIGN KEY = เชื่อม Field นี้ไปยัง Primary Key ของอีกตาราง
  ADD CONSTRAINT `fk_order_details_orders` FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`) ON DELETE CASCADE ON UPDATE CASCADE;
-- COMMIT = ยืนยันชุดการเปลี่ยนแปลงของ Transaction
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
