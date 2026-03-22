<?php 
$page_title = "My Profile";
include '../includes/header-user.php'; 

$conn = mysqli_connect("localhost", "root", "", "fintech");

$id = $_SESSION['user_id'];

// Fetch user
$checks = "SELECT * FROM signup WHERE id = '$id'";
$runs = mysqli_query($conn, $checks);
$rows = mysqli_fetch_assoc($runs);

// Wallet
$sqlp = "SELECT wallet_id FROM wallet WHERE id = '$id'";
$resultp = mysqli_query($conn, $sqlp);
$rowp = mysqli_fetch_assoc($resultp);
$wallet_id = $rowp['wallet_id'];

// UPDATE PROFILE
if (isset($_POST['update'])) {

    $fullname = $_POST['fullname'];
    $email = $_POST['email'];
    $phonenumber = $_POST['phone'];

    // IMAGE UPLOAD
    $profile_pic = $rows['profile_pic'];

    if (!empty($_FILES['profile_pic']['name'])) {
        $file_name = time() . '_' . $_FILES['profile_pic']['name'];
        $tmp_name = $_FILES['profile_pic']['tmp_name'];

        move_uploaded_file($tmp_name, "../uploads/" . $file_name);
        $profile_pic = $file_name;
    }

    $sql = "UPDATE signup SET fullname=?, email=?, phone=?, profile_pic=? WHERE id=?";
    $stmt = mysqli_prepare($conn, $sql);

    mysqli_stmt_bind_param($stmt, "ssssi",
        $fullname,
        $email,
        $phonenumber,
        $profile_pic,
        $id
    );

    mysqli_stmt_execute($stmt);

    echo "Profile updated successfully!";
}


// CHANGE PASSWORD
if (isset($_POST['change'])) {

    $curpassword = $_POST['cur_password'];
    $newpassword = $_POST['new_password'];
    $conpassword = $_POST['confirm_password'];

    $check = "SELECT * FROM signup WHERE id= '$id'";
    $run = mysqli_query($conn,$check);
    $row = mysqli_fetch_assoc($run);

    if ($curpassword != $row['passwords']) {
        echo "Current password is wrong!";
        exit();
    }

    if ($newpassword !== $conpassword) {
        echo "Passwords do not match!";
        exit();
    }

    $sql = "UPDATE signup SET passwords='$newpassword' WHERE id='$id'";
    mysqli_query($conn, $sql);

    echo "Password changed successfully!";
}
?>

<div class="grid grid-2">

    <!-- PROFILE -->
    <div class="card">
        <div class="card-header">
            <h3 class="card-title">Personal Information</h3>
        </div>

        <!-- IMAGE -->
        <div style="text-align:center; margin-bottom:24px;">
            <label for="profile_pic" style="cursor:pointer;">
                <img id="previewImage"
                     src="../uploads/<?php echo $rows['profile_pic'] ?? 'default.png'; ?>"
                     style="width:100px;height:100px;border-radius:50%;object-fit:cover;">
            </label>

            <input type="file" name="profile_pic" id="profile_pic" hidden>

            <br><br>

            <button type="button" class="btn btn-secondary btn-sm"
                onclick="document.getElementById('profile_pic').click()">
                <i class="fas fa-camera"></i> Change Photo
            </button>
        </div>

        <form method="post" enctype="multipart/form-data">

            <div class="form-group">
                <label>Full Name</label>
                <input type="text" name="fullname" class="form-control"
                       value="<?php echo $rows['fullname']; ?>" required>
            </div>

            <div class="form-group">
                <label>Email</label>
                <input type="email" name="email" class="form-control"
                       value="<?php echo $rows['email']; ?>" required>
            </div>

            <div class="form-group">
                <label>Phone</label>
                <input type="text" name="phone" class="form-control"
                       value="<?php echo $rows['phone']; ?>" required>
            </div>

            <button type="submit" name="update" class="btn btn-primary w-full">
                Update Profile
            </button>
        </form>
    </div>


    <!-- SECURITY -->
    <div class="card">
        <div class="card-header">
            <h3>Password</h3>
        </div>

        <form method="POST">

            <div class="form-group">
                <input type="password" name="cur_password" placeholder="Current Password" required>
            </div>

            <div class="form-group">
                <input type="password" name="new_password" placeholder="New Password" required>
            </div>

            <div class="form-group">
                <input type="password" name="confirm_password" placeholder="Confirm Password" required>
            </div>

            <button type="submit" name="change" class="btn btn-primary w-full">
                Change Password
            </button>
        </form>
    </div>

</div>


<!-- ACCOUNT INFO -->
<div class="card mt-4">
    <h3>Account Info</h3>

    <p>User ID: <?php echo $id; ?></p>
    <p>Wallet ID: <?php echo $wallet_id; ?></p>
    <p>Member Since: <?php echo $rows['joined_at']; ?></p>
</div>


<!-- PREVIEW SCRIPT -->
<script>
document.getElementById('profile_pic').addEventListener('change', function(e) {
    const file = e.target.files[0];
    if (file) {
        document.getElementById('previewImage').src = URL.createObjectURL(file);
    }
});
</script>

<?php include '../includes/footer.php'; ?>
