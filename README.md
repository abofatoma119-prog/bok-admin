<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة أرصدة BOK</title>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --primary: #0056b3; --accent: #e67e22; --bg: #f0f2f5; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); display: flex; justify-content: center; padding: 20px; }
        .container { background: white; width: 100%; max-width: 450px; padding: 25px; border-radius: 20px; box-shadow: 0 15px 35px rgba(0,0,0,0.1); }
        .header { text-align: center; border-bottom: 2px solid #eee; margin-bottom: 20px; padding-bottom: 10px; }
        .input-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: 600; color: #333; }
        input { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 10px; box-sizing: border-box; font-size: 16px; transition: 0.3s; }
        input:focus { border-color: var(--primary); outline: none; }
        .balance-preview { background: #eef6ff; padding: 15px; border-radius: 10px; margin-bottom: 15px; display: none; border-right: 5px solid var(--primary); }
        .btn { width: 100%; padding: 14px; border-radius: 10px; border: none; font-weight: bold; cursor: pointer; font-size: 16px; transition: 0.3s; margin-top: 10px; }
        .btn-check { background: var(--accent); color: white; }
        .btn-update { background: var(--primary); color: white; display: none; }
        .status { margin-top: 15px; padding: 10px; border-radius: 8px; text-align: center; font-size: 14px; display: none; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h2 style="color: var(--primary);">تحديث رصيد الحساب</h2>
        <p style="font-size: 12px; color: #666;">مشروع: ontest-92fe9</p>
    </div>

    <div class="input-group">
        <label>رقم الحساب (16 رقم)</label>
        <input type="text" id="acc_num" placeholder="أدخل رقم الحساب هنا">
        <button class="btn btn-check" onclick="checkUser()">فحص الحساب</button>
    </div>

    <div id="preview" class="balance-preview">
        <div style="font-size: 13px; color: #555;">الرصيد الحالي:</div>
        <div id="current_bal_val" style="font-size: 24px; font-weight: bold; color: var(--primary);">0.00</div>
    </div>

    <div id="update-section" style="display: none;">
        <div class="input-group">
            <label>المبلغ المراد إضافته (+)</label>
            <input type="number" id="amount_to_add" placeholder="مثلاً: 5000">
        </div>
        <button class="btn btn-update" id="upd-btn" style="display: block;" onclick="updateBalance()">تأكيد زيادة الرصيد</button>
    </div>

    <div id="msg" class="status"></div>
</div>

<script>
    const firebaseConfig = {
        apiKey: "AIzaSyBR1lzuYqNUg1yRRdiNau4o6rjzpX8GjRo",
        authDomain: "ontest-92fe9.firebaseapp.com",
        databaseURL: "https://ontest-92fe9-default-rtdb.firebaseio.com",
        projectId: "ontest-92fe9",
        appId: "1:629491598570:android:fa1376f70fd64dd0c30393"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    let currentUserKey = null;
    let currentBalance = 0;

    function checkUser() {
        const acc = document.getElementById('acc_num').value;
        if(!acc) return alert("أدخل رقم الحساب");

        db.ref('Users').orderByChild('account_number').equalTo(acc).once('value', (s) => {
            if(s.exists()) {
                s.forEach(child => {
                    currentUserKey = child.key;
                    currentBalance = parseFloat(child.val().balance) || 0;
                    document.getElementById('current_bal_val').innerText = currentBalance.toLocaleString() + " SDG";
                    document.getElementById('preview').style.display = 'block';
                    document.getElementById('update-section').style.display = 'block';
                    showMsg("تم العثور على الحساب", false);
                });
            } else {
                showMsg("الحساب غير موجود!", true);
                document.getElementById('preview').style.display = 'none';
                document.getElementById('update-section').style.display = 'none';
            }
        });
    }

    function updateBalance() {
        const add = parseFloat(document.getElementById('amount_to_add').value);
        if(isNaN(add) || add <= 0) return alert("أدخل مبلغاً صحيحاً");

        const newBal = currentBalance + add;
        db.ref('Users/' + currentUserKey).update({ balance: newBal })
        .then(() => {
            showMsg("تم التحديث بنجاح!");
            checkUser(); // لتحديث العرض
            document.getElementById('amount_to_add').value = "";
        });
    }

    function showMsg(m, isErr) {
        const el = document.getElementById('msg');
        el.innerText = m;
        el.style.display = 'block';
        el.style.background = isErr ? '#ffdada' : '#d4edda';
        el.style.color = isErr ? '#a94442' : '#155724';
    }
</script>
</body>
</html>
