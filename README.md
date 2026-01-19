<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Toki Pro | Global Hub</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, orderBy, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";
        import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-storage.js";

        // Your exact Firebase Config
        const firebaseConfig = {
            apiKey: "AIzaSyANqGXgK_cPaF0h6xcn75J-qoM_WCvBXaw",
            authDomain: "toki-c3e68.firebaseapp.com",
            projectId: "toki-c3e68",
            storageBucket: "toki-c3e68.firebasestorage.app", 
            messagingSenderId: "881327306815",
            appId: "1:881327306815:web:19f1c22beb8d699326208a"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const storage = getStorage(app);

        async function initApp() {
            let uid = localStorage.getItem('tokiUser') || "user_" + Date.now();
            localStorage.setItem('tokiUser', uid);
            const userRef = doc(db, "creators", uid);
            const snap = await getDoc(userRef);
            if (!snap.exists()) {
                let name = prompt("Enter your Stage Name:") || "Toki Star";
                await setDoc(userRef, { name: name, balance: 0, views: 0 });
            }
            loadFeed();
            updateBalance();
        }

        async function updateBalance() {
            const snap = await getDoc(doc(db, "creators", localStorage.getItem('tokiUser')));
            document.getElementById('balanceText').innerText = "$" + snap.data().balance.toFixed(2);
        }

        window.publishNow = async () => {
            const file = document.getElementById('mainFile').files[0];
            const title = document.getElementById('vTitle').value;
            const category = document.getElementById('vCat').value;
            
            if (!file || !title) return alert("Please add a file and title!");

            document.getElementById('loader').classList.remove('hidden');

            try {
                const sRef = ref(storage, 'toki/' + Date.now() + "_" + file.name);
                const task = await uploadBytes(sRef, file);
                const url = await getDownloadURL(task.ref);

                await addDoc(collection(db, "toki_feed"), {
                    title: title,
                    type: category,
                    url: url,
                    author: (await getDoc(doc(db, "creators", localStorage.getItem('tokiUser')))).data().name,
                    authorId: localStorage.getItem('tokiUser'),
                    views: 0,
                    time: new Date()
                });

                alert("Successfully Uploaded! 🚀");
                document.getElementById('uploadUI').classList.add('hidden');
            } catch (e) {
                alert("Upload Failed! Check your Firebase Storage Rules.");
            }
            document.getElementById('loader').classList.add('hidden');
        };

        function loadFeed() {
            onSnapshot(query(collection(db, "toki_feed"), orderBy("time", "desc")), (snap) => {
                const feed = document.getElementById('feed');
                feed.innerHTML = '';
                snap.forEach(d => {
                    const post = d.data();
                    let media = post.type === 'video' ? 
                        `<video src="${post.url}" class="w-full rounded-2xl shadow-xl" controls></video>` : 
                        `<img src="${post.url}" class="w-full rounded-2xl shadow-xl">`;
                    
                    feed.innerHTML += `
                        <div class="mb-10 bg-[#121212] p-4 rounded-[30px] border border-gray-800 animate-fade-in">
                            <div class="flex items-center gap-3 mb-4">
                                <div class="w-10 h-10 bg-gradient-to-tr from-red-600 to-purple-600 rounded-full flex items-center justify-center font-bold">T</div>
                                <div>
                                    <h4 class="font-bold text-sm">@${post.author}</h4>
                                    <p class="text-[10px] text-gray-500 uppercase">${post.type}</p>
                                </div>
                            </div>
                            <h2 class="text-lg font-semibold mb-4 text-white/90">${post.title}</h2>
                            ${media}
                            <div class="flex gap-6 mt-5 text-gray-400">
                                <i class="far fa-heart text-xl hover:text-red-500 transition"></i>
                                <i class="far fa-comment text-xl hover:text-blue-500 transition"></i>
                                <i class="fas fa-share text-xl hover:text-green-500 transition"></i>
                            </div>
                        </div>`;
                });
            });
        }

        window.onload = initApp;
    </script>

    <style>
        @keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-fade-in { animation: fade-in 0.5s ease-out; }
        ::-webkit-scrollbar { display: none; }
        .glass { background: rgba(255, 255, 255, 0.03); backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.05); }
    </style>
</head>
<body class="bg-[#050505] text-white font-sans selection:bg-red-600 selection:text-white">

    <header class="fixed top-0 w-full z-50 glass px-6 py-4 flex justify-between items-center">
        <h1 class="text-2xl font-black italic tracking-tighter text-red-600">TOKI <span class="text-white not-italic">PRO</span></h1>
        <div class="bg-red-600/10 border border-red-600/30 px-4 py-1 rounded-full">
            <span id="balanceText" class="text-red-500 font-black text-sm">$ 0.00</span>
        </div>
    </header>

    <div id="loader" class="hidden fixed inset-0 bg-black/95 z-[100] flex items-center justify-center">
        <div class="text-center">
            <div class="w-16 h-16 border-4 border-red-600 border-t-transparent rounded-full animate-spin mx-auto mb-4"></div>
            <p class="font-black tracking-widest uppercase text-sm">Uploading to Galaxy...</p>
        </div>
    </div>

    <main id="feed" class="max-w-md mx-auto pt-28 px-4 pb-32"></main>

    <div id="uploadUI" class="fixed inset-0 bg-black/90 hidden z-[60] p-6 flex items-center justify-center">
        <div class="bg-[#111] border border-gray-800 w-full rounded-[40px] p-8 shadow-2xl">
            <h2 class="text-2xl font-black mb-6 text-center italic">NEW CONTENT</h2>
            <input type="text" id="vTitle" placeholder="Title of your story..." class="w-full bg-[#181818] p-4 rounded-2xl mb-4 border border-gray-800 outline-none focus:border-red-600">
            <select id="vCat" class="w-full bg-[#181818] p-4 rounded-2xl mb-4 border border-gray-800 outline-none">
                <option value="video">Short/Long Video</option>
                <option value="photo">Beautiful Photo</option>
            </select>
            <input type="file" id="mainFile" class="w-full mb-6 text-sm text-gray-500 file:bg-white file:text-black file:border-none file:px-6 file:py-3 file:rounded-xl file:font-black">
            <div class="flex gap-3">
                <button onclick="publishNow()" class="flex-2 bg-red-600 text-white w-full py-4 rounded-2xl font-black shadow-lg shadow-red-600/20 active:scale-95 transition">PUBLISH</button>
                <button onclick="document.getElementById('uploadUI').classList.add('hidden')" class="flex-1 bg-gray-800 text-white w-full py-4 rounded-2xl font-bold">CLOSE</button>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 w-full bg-black/80 backdrop-blur-xl border-t border-white/5 flex justify-around p-6 items-center z-50">
        <i class="fas fa-home text-2xl text-red-600"></i>
        <i class="fas fa-search text-2xl text-gray-600"></i>
        <button onclick="document.getElementById('uploadUI').classList.remove('hidden')" class="w-16 h-12 bg-white text-black rounded-2xl flex items-center justify-center shadow-xl active:scale-90 transition">
            <i class="fas fa-plus text-xl"></i>
        </button>
        <i class="fas fa-bolt text-2xl text-gray-600"></i>
        <i class="fas fa-user text-2xl text-gray-600"></i>
    </nav>

</body>
</html>
