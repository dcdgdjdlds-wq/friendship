<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Friendship Hub - บันทึกภาพและความทรงจำ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Kanit', sans-serif; background-color: #fff5ed; }
        .message-card {
            background: white;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
            margin-bottom: 20px;
            padding: 0;
            border-radius: 20px;
            overflow: hidden;
        }
        .hidden { display: none; }
        .active-tab { border-bottom: 4px solid #f97316; color: #f97316; }
    </style>
</head>
<body class="p-0 md:p-10">

    <div class="max-w-2xl mx-auto bg-white min-h-screen md:min-h-0 md:rounded-3xl shadow-2xl overflow-hidden">
        <div class="flex border-b bg-white sticky top-0 z-10">
            <button onclick="showPage('write')" id="tabWrite" class="flex-1 py-4 font-bold active-tab">✍️ เขียนและลงรูป</button>
            <button onclick="showPage('view')" id="tabView" class="flex-1 py-4 font-bold text-gray-400">📖 ดูสมุดบันทึก</button>
        </div>

        <div id="pageWrite" class="p-6">
            <h2 class="text-2xl font-bold text-gray-800 mb-6 text-center">บันทึกความทรงจำใหม่</h2>
            <form id="friendForm" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">ชื่อเพื่อน / ผู้ส่ง</label>
                    <input type="text" id="nameInput" placeholder="ใครเป็นคนเขียนนะ..." class="w-full p-4 border-2 border-orange-50 rounded-2xl outline-none focus:border-orange-500 bg-orange-50/30" required>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">ความรู้สึก/ข้อความ</label>
                    <textarea id="msgInput" placeholder="เล่าเรื่องราวที่นี่..." rows="3" class="w-full p-4 border-2 border-orange-50 rounded-2xl outline-none focus:border-orange-500 bg-orange-50/30" required></textarea>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">แนบรูปภาพความทรงจำ</label>
                    <input type="file" id="imageInput" accept="image/*" class="w-full p-2 text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-orange-50 file:text-orange-700 hover:file:bg-orange-100">
                </div>
                <button type="submit" class="w-full bg-orange-500 text-white font-bold py-4 rounded-2xl hover:bg-orange-600 transition shadow-lg text-lg">
                    บันทึกความทรงจำ ✨
                </button>
            </form>
        </div>

        <div id="pageView" class="p-6 hidden">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-gray-800">สมุดบันทึกภาพ</h2>
                <button onclick="clearAll()" class="text-xs text-red-400 hover:text-red-600">ล้างทั้งหมด</button>
            </div>
            
            <div id="messagesContainer" class="grid grid-cols-1 gap-6">
                </div>
        </div>
    </div>

    <script>
        const container = document.getElementById('messagesContainer');
        const form = document.getElementById('friendForm');

        function showPage(page) {
            if(page === 'write') {
                document.getElementById('pageWrite').classList.remove('hidden');
                document.getElementById('pageView').classList.add('hidden');
                document.getElementById('tabWrite').classList.add('active-tab');
                document.getElementById('tabView').classList.remove('active-tab');
                document.getElementById('tabView').classList.add('text-gray-400');
            } else {
                document.getElementById('pageWrite').classList.add('hidden');
                document.getElementById('pageView').classList.remove('hidden');
                document.getElementById('tabWrite').classList.remove('active-tab');
                document.getElementById('tabWrite').classList.add('text-gray-400');
                document.getElementById('tabView').classList.add('active-tab');
                displayMessages();
            }
        }

        // ฟังก์ชันแปลงรูปภาพเป็นข้อความ (Base64) เพื่อเก็บในเครื่อง
        function getBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
            });
        }

        form.addEventListener('submit', async function(e) {
            e.preventDefault();
            const name = document.getElementById('nameInput').value;
            const text = document.getElementById('msgInput').value;
            const imageFile = document.getElementById('imageInput').files[0];
            
            let imageData = "";
            if (imageFile) {
                imageData = await getBase64(imageFile);
            }

            const saved = JSON.parse(localStorage.getItem('photo_msgs')) || [];
            saved.unshift({ name, text, image: imageData, date: new Date().toLocaleDateString() });
            localStorage.setItem('photo_msgs', JSON.stringify(saved));
            
            alert('บันทึกสำเร็จ!');
            form.reset();
            showPage('view');
        });

        function displayMessages() {
            const saved = JSON.parse(localStorage.getItem('photo_msgs')) || [];
            if (saved.length === 0) {
                container.innerHTML = `<p class="text-center text-gray-400 py-20">ยังไม่มีบันทึก... ลองเพิ่มรูปแรกสิ!</p>`;
                return;
            }

            container.innerHTML = saved.map((m, index) => `
                <div class="message-card border border-orange-100">
                    ${m.image ? `<img src="${m.image}" class="w-full h-64 object-cover">` : ''}
                    <div class="p-5">
                        <div class="flex justify-between items-center mb-2">
                            <h3 class="font-bold text-orange-600 text-lg">${m.name}</h3>
                            <span class="text-xs text-gray-400">${m.date}</span>
                        </div>
                        <p class="text-gray-700 italic">"${m.text}"</p>
                        <button onclick="deleteMsg(${index})" class="mt-4 text-xs text-red-300 hover:text-red-500">ลบบันทึกนี้</button>
                    </div>
                </div>
            `).join('');
        }

        function deleteMsg(i) {
            if(confirm('ลบความทรงจำนี้ใช่ไหม?')) {
                let saved = JSON.parse(localStorage.getItem('photo_msgs'));
                saved.splice(i, 1);
                localStorage.setItem('photo_msgs', JSON.stringify(saved));
                displayMessages();
            }
        }

        function clearAll() {
            if(confirm('ล้างสมุดบันทึกทั้งหมด?')) {
                localStorage.removeItem('photo_msgs');
                displayMessages();
            }
        }
    </script>
</body>
</html>
