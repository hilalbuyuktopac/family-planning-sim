<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hemşire Akademisi - Aile Planlaması Oyunu</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        @keyframes slideIn {
            from { transform: translateX(100%); }
            to { transform: translateX(0); }
        }
        .animate-fadeIn { animation: fadeIn 0.5s ease-out; }
        .animate-slideIn { animation: slideIn 0.3s ease-out; }
        .hover-scale:hover { transform: scale(1.02); }
        .hover-scale-105:hover { transform: scale(1.05); }
    </style>
</head>
<body class="bg-gray-100">
    <div id="app"></div>

    <script>
        const gameData = {
            gameState: 'intro',
            playerName: '',
            currentDay: 1,
            energy: 100,
            reputation: 0,
            coins: 0,
            inventory: [],
            badges: [],
            completedPatients: [],
            currentPatient: null,
            dialogueStep: 0
        };

        const patients = [
            {
                id: 1,
                name: "Ayşe & Mehmet",
                avatar: "👫",
                mood: "😊",
                urgency: "normal",
                coins: 150,
                background: "from-pink-100 to-purple-100",
                story: "Yeni evli bir çift. Rüya gibi bir balayı geçirdiler ve şimdi kariyerlerine odaklanmak istiyorlar.",
                dialogue: [
                    {
                        text: "Merhaba hemşire! Biz yeni evlendik ve 2-3 yıl bebek sahibi olmayı ertelemek istiyoruz.",
                        choices: [
                            { text: "🎉 Önce tebrikler! Size yardımcı olmaktan mutluluk duyarım.", correct: true, response: "Teşekkür ederiz! Çok heyecanlıyız ama biraz da karışık durumdayız.", points: 20 },
                            { text: "Tamam, hangi yöntemi istiyorsunuz?", correct: false, response: "Aslında... bilmiyoruz. Bu yüzden size geldik.", points: 0 }
                        ]
                    },
                    {
                        text: "Düzenli bir yaşamımız var, her gün ilaç almayı unutmayız. Sizce ne önerirsiniz?",
                        choices: [
                            { text: "💊 Kombine oral kontraseptif (hap) sizin için ideal olabilir!", correct: true, response: "Harika! Peki nasıl kullanacağız?", points: 30 },
                            { text: "🔪 Kalıcı yöntem olan tüp ligasyonu düşünün.", correct: false, response: "Ama biz gelecekte çocuk istiyoruz...", points: -10 }
                        ]
                    },
                    {
                        text: "Her gün aynı saatte mi almalıyız? Ya unutursak?",
                        choices: [
                            { text: "⏰ Evet, her gün aynı saatte alın. Telefon alarmı kurun. 12 saatten fazla gecikme olursa ek koruma kullanın.", correct: true, response: "Anladık! Çok net anlattınız, teşekkürler!", points: 30, badge: "İletişim Uzmanı" },
                            { text: "Önemli değil, istediğiniz zaman alabilirsiniz.", correct: false, response: "Hmm, emin misiniz? İnternette farklı okumuştuk...", points: -20 }
                        ]
                    }
                ]
            },
            {
                id: 2,
                name: "Zeynep",
                avatar: "🤱",
                mood: "😰",
                urgency: "high",
                coins: 200,
                background: "from-blue-100 to-green-100",
                story: "3 aylık bebeğini emziren genç bir anne. Yorgun ama bebeğine çok özen gösteriyor.",
                dialogue: [
                    {
                        text: "Hemşire hanım, çok yorgunum... 3 aylık bebeğim var ve emziriyorum. Tekrar hamile kalmaktan korkuyorum.",
                        choices: [
                            { text: "❤️ Anladım, hem sizi hem bebeğinizi düşünerek bir çözüm bulalım.", correct: true, response: "Çok teşekkür ederim... Bebeğime zarar vermeyecek bir şey olmalı.", points: 20 },
                            { text: "Zaten emziriyorsunuz, hamile kalamazsınız.", correct: false, response: "Gerçekten mi? Ama komşum emzirirken hamile kaldı...", points: -15 }
                        ]
                    },
                    {
                        text: "Doğum kontrol hapı alabilir miyim? Anne sütümü etkiler mi?",
                        choices: [
                            { text: "🍼 Mini pill (sadece progesteron) alabilirsiniz. Bebek için güvenli!", correct: true, response: "Ah ne güzel! Nasıl kullanmalıyım?", points: 30 },
                            { text: "Kombine hap (östrojen+progesteron) kullanın.", correct: false, response: "Ama östrojen süt üretimini azaltmaz mı?", points: -10 }
                        ]
                    },
                    {
                        text: "LAM diye bir şey duydum, o nedir?",
                        choices: [
                            { text: "📋 LAM: Bebek 6 aydan küçük, SADECE anne sütü, adet görmediniz - bu 3'ü varsa %98 korur!", correct: true, response: "Vay be, bilmiyordum! Çok bilgilendirici oldu.", points: 30, badge: "Bilgi Hazinesi" },
                            { text: "Emzirme tek başına yeterli korumadır.", correct: false, response: "Peki ya adetim gelirse? Hala korur mu?", points: -20 }
                        ]
                    }
                ]
            },
            {
                id: 3,
                name: "Elif",
                avatar: "👧",
                mood: "😳",
                urgency: "high",
                coins: 180,
                background: "from-yellow-100 to-orange-100",
                story: "18 yaşında üniversite öğrencisi. Utangaç ama sorularına cevap bulmak istiyor.",
                dialogue: [
                    {
                        text: "*utangaç* Merhaba... Arkadaşım gönderdi beni. Korunma yöntemleri hakkında bilgi almak istiyorum.",
                        choices: [
                            { text: "😊 Hoş geldin Elif. Burası güvenli bir alan, sorularını rahatça sorabilirsin. Gizliliğine saygı duyuyorum.", correct: true, response: "*rahatlar* Teşekkür ederim... Biraz gerginim.", points: 25 },
                            { text: "Ailene söyledin mi geldiğini?", correct: false, response: "*paniğe kapılır* Hayır! Onlara söyleyecek misiniz?", points: -20 }
                        ]
                    },
                    {
                        text: "Sadece hamile kalmamak mı yeterli? Başka nelerden korunmalıyım?",
                        choices: [
                            { text: "🛡️ Hem gebelik hem de cinsel yolla bulaşan enfeksiyonlardan (CYBE) korunman önemli!", correct: true, response: "Anladım... Peki her ikisinden birden nasıl korunurum?", points: 25 },
                            { text: "Sadece hamilelik önemli, gerisini düşünme.", correct: false, response: "Ama internette başka şeyler okudum...", points: -15 }
                        ]
                    },
                    {
                        text: "En iyi yöntem nedir benim için?",
                        choices: [
                            { text: "🎯 Çift koruma: Kondom (CYBE'den korur) + doğum kontrol hapı (gebeliği önler). İkisi birlikte en güvenli!", correct: true, response: "Vay! Bunu hiç düşünmemiştim. Çok mantıklı!", points: 35, badge: "Koruyucu Melek" },
                            { text: "Geri çekme yöntemi yeterlidir.", correct: false, response: "Arkadaşım öyle yaptı ve hamileydi... Başka seçenek yok mu?", points: -25 }
                        ]
                    }
                ]
            }
        ];

        const shopItems = [
            { id: 1, name: "☕ Kahve", cost: 30, energy: 20, icon: "☕" },
            { id: 2, name: "📚 Ders Kitabı", cost: 100, reputation: 50, icon: "📚" },
            { id: 3, name: "🎧 Müzik", cost: 50, energy: 30, icon: "🎧" },
            { id: 4, name: "💪 Spor", cost: 40, energy: 25, icon: "💪" }
        ];

        function showNotification(message, type = 'success') {
            const notification = document.createElement('div');
            notification.className = `fixed top-6 right-6 px-6 py-4 rounded-xl shadow-2xl z-50 animate-slideIn ${
                type === 'success' ? 'bg-green-500 text-white' :
                type === 'badge' ? 'bg-yellow-500 text-white' :
                'bg-red-500 text-white'
            }`;
            notification.textContent = message;
            document.body.appendChild(notification);
            setTimeout(() => notification.remove(), 3000);
        }

        function renderIntro() {
            return `
                <div class="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 flex items-center justify-center p-6">
                    <div class="max-w-2xl w-full bg-white rounded-3xl shadow-2xl p-12 text-center animate-fadeIn">
                        <div class="text-8xl mb-6">🏥</div>
                        <h1 class="text-5xl font-bold bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent mb-4">
                            Hemşire Akademisi
                        </h1>
                        <p class="text-xl text-gray-600 mb-8">Aile Planlaması Danışmanlığı Macerası</p>
                        
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 rounded-2xl p-8 mb-8">
                            <h2 class="text-2xl font-bold text-gray-800 mb-4">🎮 Oyun Nasıl Oynanır?</h2>
                            <div class="text-left space-y-3 text-gray-700">
                                <p>✨ Hastanede farklı hastalarla konuş</p>
                                <p>💬 Doğru danışmanlık yap ve itibar kazan</p>
                                <p>⚡ Enerjini yönet, dinlen ve alışveriş yap</p>
                                <p>🏆 Rozetler topla ve uzman hemşire ol!</p>
                            </div>
                        </div>

                        <input
                            type="text"
                            id="playerNameInput"
                            placeholder="Hemşire adını gir..."
                            class="w-full px-6 py-4 text-lg border-3 border-purple-300 rounded-xl mb-6 focus:border-purple-500 focus:outline-none"
                        />
                        
                        <button
                            onclick="startGame()"
                            class="w-full py-4 bg-gradient-to-r from-purple-600 to-pink-600 text-white text-xl font-bold rounded-xl hover:from-purple-700 hover:to-pink-700 transition-all hover-scale-105"
                        >
                            🚀 Maceraya Başla!
                        </button>
                    </div>
                </div>
            `;
        }

        function renderHospital() {
            const level = Math.floor(gameData.reputation / 100) + 1;
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 via-purple-50 to-pink-50 p-6">
                    <div class="max-w-7xl mx-auto">
                        <div class="bg-white rounded-3xl shadow-2xl p-6 mb-6">
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div>
                                    <h1 class="text-3xl font-bold text-gray-800">Hemşire ${gameData.playerName}</h1>
                                    <p class="text-gray-600">Gün ${gameData.currentDay} • ${gameData.badges.length} Rozet</p>
                                </div>
                                
                                <div class="flex gap-4 flex-wrap">
                                    <div class="bg-gradient-to-br from-red-100 to-pink-100 px-6 py-3 rounded-xl">
                                        <div class="text-sm text-gray-600">⚡ Enerji</div>
                                        <div class="text-2xl font-bold text-red-600">${gameData.energy}/100</div>
                                    </div>
                                    
                                    <div class="bg-gradient-to-br from-yellow-100 to-orange-100 px-6 py-3 rounded-xl">
                                        <div class="text-sm text-gray-600">⭐ İtibar</div>
                                        <div class="text-2xl font-bold text-yellow-600">${gameData.reputation}</div>
                                    </div>
                                    
                                    <div class="bg-gradient-to-br from-green-100 to-emerald-100 px-6 py-3 rounded-xl">
                                        <div class="text-sm text-gray-600">💰 Para</div>
                                        <div class="text-2xl font-bold text-green-600">${gameData.coins}</div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="grid md:grid-cols-3 gap-6">
                            <div class="md:col-span-2 bg-white rounded-3xl shadow-2xl p-8">
                                <h2 class="text-2xl font-bold text-gray-800 mb-6">👥 Bekleme Odası</h2>
                                <div class="space-y-4">
                                    ${patients.map(patient => `
                                        <div class="bg-gradient-to-br ${patient.background} rounded-2xl p-6 border-2 border-opacity-50 hover:border-opacity-100 transition-all hover-scale">
                                            <div class="flex items-center justify-between flex-wrap gap-4">
                                                <div class="flex items-center gap-4">
                                                    <div class="text-6xl">${patient.avatar}</div>
                                                    <div>
                                                        <h3 class="text-xl font-bold text-gray-800">${patient.name}</h3>
                                                        <p class="text-sm text-gray-600 mb-2">${patient.story}</p>
                                                        <div class="flex items-center gap-3">
                                                            <span class="text-2xl">${patient.mood}</span>
                                                            <span class="px-3 py-1 rounded-full text-xs font-bold ${patient.urgency === 'high' ? 'bg-red-200 text-red-800' : 'bg-green-200 text-green-800'}">
                                                                ${patient.urgency === 'high' ? '🔥 Acil' : '✓ Normal'}
                                                            </span>
                                                            <span class="text-sm font-semibold text-green-700">+${patient.coins} 💰</span>
                                                        </div>
                                                    </div>
                                                </div>
                                                <button
                                                    onclick="startPatientVisit(${patient.id})"
                                                    ${gameData.energy < 20 ? 'disabled' : ''}
                                                    class="px-6 py-3 bg-gradient-to-r from-blue-600 to-purple-600 text-white font-bold rounded-xl hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed transition-all hover-scale-105"
                                                >
                                                    Görüş
                                                </button>
                                            </div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>

                            <div class="space-y-6">
                                <div class="bg-white rounded-3xl shadow-2xl p-6">
                                    <h2 class="text-xl font-bold text-gray-800 mb-4">🏪 Mağaza</h2>
                                    <div class="space-y-3">
                                        ${shopItems.map(item => `
                                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 rounded-xl p-4">
                                                <div class="flex items-center justify-between mb-2">
                                                    <span class="font-bold text-gray-800">${item.name}</span>
                                                    <span class="text-2xl">${item.icon}</span>
                                                </div>
                                                <div class="flex items-center justify-between">
                                                    <span class="text-sm text-gray-600">${item.cost} 💰</span>
                                                    <button
                                                        onclick="buyItem(${item.id})"
                                                        ${gameData.coins < item.cost ? 'disabled' : ''}
                                                        class="px-4 py-2 bg-purple-600 text-white text-sm font-bold rounded-lg hover:bg-purple-700 disabled:opacity-50 disabled:cursor-not-allowed transition-all"
                                                    >
                                                        Satın Al
                                                    </button>
                                                </div>
                                            </div>
                                        `).join('')}
                                    </div>
                                </div>

                                ${gameData.badges.length > 0 ? `
                                    <div class="bg-white rounded-3xl shadow-2xl p-6">
                                        <h2 class="text-xl font-bold text-gray-800 mb-4">🏆 Rozetlerim</h2>
                                        <div class="space-y-2">
                                            ${gameData.badges.map(badge => `
                                                <div class="bg-gradient-to-r from-yellow-50 to-orange-50 rounded-xl p-3 flex items-center gap-3">
                                                    <span class="text-2xl">🏆</span>
                                                    <span class="font-semibold text-gray-800">${badge}</span>
                                                </div>
                                            `).join('')}
                                        </div>
                                    </div>
                                ` : ''}
                            </div>
                        </div>
                    </div>
                </div>
            `;
        }

        function renderConsultation() {
            const patient = gameData.currentPatient;
            const dialogue = patient.dialogue[gameData.dialogueStep];
            
            return `
                <div class="min-h-screen bg-gradient-to-br ${patient.background} p-6 flex items-center justify-center">
                    <div class="max-w-4xl w-full">
                        <div class="bg-white rounded-3xl shadow-2xl p-8 mb-6 animate-fadeIn">
                            <div class="flex items-center gap-4 mb-6">
                                <div class="text-7xl">${patient.avatar}</div>
                                <div>
                                    <h2 class="text-3xl font-bold text-gray-800">${patient.name}</h2>
                                    <p class="text-gray-600">Muayene Odası • Adım ${gameData.dialogueStep + 1}/${patient.dialogue.length}</p>
                                </div>
                            </div>

                            <div class="bg-gray-100 rounded-2xl p-6 mb-6">
                                <p class="text-lg text-gray-800 leading-relaxed">${dialogue.text}</p>
                            </div>

                            <div class="space-y-3">
                                ${dialogue.choices.map((choice, idx) => `
                                    <button
                                        onclick="handleChoice(${idx})"
                                        class="w-full p-6 bg-gradient-to-r from-blue-50 to-purple-50 hover:from-blue-100 hover:to-purple-100 rounded-2xl text-left transition-all hover-scale border-2 border-transparent hover:border-blue-400"
                                    >
                                        <p class="text-lg text-gray-800">${choice.text}</p>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <div class="flex gap-4 justify-center flex-wrap">
                            <div class="bg-white px-6 py-3 rounded-xl shadow-lg">
                                <span class="font-semibold text-gray-700">⏰ Gün ${gameData.currentDay}</span>
                            </div>
                            <div class="bg-white px-6 py-3 rounded-xl shadow-lg">
                                <span class="font-semibold text-gray-700">⭐ ${gameData.reputation} İtibar</span>
                            </div>
                        </div>
                    </div>
                </div>
            `;
        }

        function render() {
            const app = document.getElementById('app');
            if (gameData.gameState === 'intro') {
                app.innerHTML = renderIntro();
            } else if (gameData.gameState === 'hospital') {
                app.innerHTML = renderHospital();
            } else if (gameData.gameState === 'consultation') {
                app.innerHTML = renderConsultation();
            }
        }

        function startGame() {
            const input = document.getElementById('playerNameInput');
            if (input.value.trim()) {
                gameData.playerName = input.value.trim();
                gameData.gameState = 'hospital';
                render();
            }
        }

        function startPatientVisit(patientId) {
            if (gameData.energy < 20) {
                showNotification('😴 Çok yorgunsun! Önce dinlen.', 'error');
                return;
            }
            gameData.currentPatient = patients.find(p => p.id === patientId);
            gameData.dialogueStep = 0;
            gameData.energy -= 20;
            gameData.gameState = 'consultation';
            render();
        }

        function handleChoice(choiceIdx) {
            const dialogue = gameData.currentPatient.dialogue[gameData.dialogueStep];
            const choice = dialogue.choices[choiceIdx];
            
            if (choice.correct) {
                gameData.reputation += choice.points;
                showNotification(`+${choice.points} İtibar! 🌟`, 'success');
                
                if (choice.badge && !gameData.badges.includes(choice.badge)) {
                    gameData.badges.push(choice.badge);
                    showNotification(`🏆 Yeni Rozet: ${choice.badge}`, 'badge');
                }
            } else {
                if (choice.points < 0) {
                    gameData.reputation = Math.max(0, gameData.reputation + choice.points);
                    showNotification(`${choice.points} İtibar 😕`, 'error');
                }
            }

            setTimeout(() => {
                if (gameData.dialogueStep < gameData.currentPatient.dialogue.length - 1) {
                    gameData.dialogueStep++;
                    render();
                } else {
                    gameData.coins += gameData.currentPatient.coins;
                    showNotification(`Tamamlandı! +${gameData.currentPatient.coins} 💰`, 'success');
                    gameData.currentDay++;
                    gameData.completedPatients.push(gameData.currentPatient.id);
                    gameData.currentPatient = null;
                    gameData.gameState = 'hospital';
                    render();
                }
            }, 1500);
        }

        function buyItem(itemId) {
            const item = shopItems.find(i => i.id === itemId);
            if (gameData.coins >= item.cost) {
                gameData.coins -= item.cost;
                if (item.energy) gameData.energy = Math.min(100, gameData.energy + item.energy);
                if (item.reputation) gameData.reputation += item.reputation;
                gameData.inventory.push(item.name);
                showNotification(`${item.icon} Satın alındı!`, 'success');
                render();
            } else {
                showNotification('Yeterli paran yok! 💸', 'error');
            }
        }

        // İlk render
        render();
    </script>
</body>
</html>
