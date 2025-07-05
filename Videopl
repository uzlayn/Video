<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0" />
  <title>Улучшенный видео плеер</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    :root {
      --primary-color: #3B82F6;
      --tg-theme-bg-color: var(--tg-bg-color, #111827);
      --tg-theme-text-color: var(--tg-text-color, #F9FAFB);
      --tg-theme-hint-color: var(--tg-hint-color, #9CA3AF);
      --tg-theme-link-color: var(--tg-link-color, #3B82F6);
      --tg-theme-button-color: var(--tg-button-color, #3B82F6);
      --tg-theme-button-text-color: var(--tg-button-text-color, #F9FAFB);
      --tg-theme-secondary-bg-color: var(--tg-secondary-bg-color, #1F2937);
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; overflow: hidden; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      background-color: var(--tg-theme-bg-color);
      color: var(--tg-theme-text-color);
      display: flex;
      flex-direction: column;
      user-select: none;
    }
    .main-container { width: 100%; height: 100%; display: flex; flex-direction: column; max-width: 800px; margin: 0 auto; }
    #player-section { flex-shrink: 0; }
    .video-player-wrapper { position: relative; background-color: #000; aspect-ratio: 16 / 9; border-radius: 12px; overflow: hidden; }
    #main-video { width: 100%; height: 100%; display: block; }

    .controls-overlay {
      position: absolute; inset: 0; display: flex; flex-direction: column; justify-content: flex-end;
      background: linear-gradient(to top, rgba(0,0,0,0.65) 0%, rgba(0,0,0,0.3) 30%, transparent 50%);
      opacity: 0; transition: opacity 0.3s ease-out; padding: 0.75rem 1.25rem; pointer-events: none;
    }
    .video-player-wrapper:hover .controls-overlay, .video-player-wrapper.paused-state .controls-overlay { opacity: 1; pointer-events: auto; }
    .controls-overlay > * { pointer-events: auto; }
    .center-controls { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
    .icon-btn {
      background: none; border: none; color: var(--tg-theme-text-color); font-size: 1.2rem; cursor: pointer;
      padding: 0.5rem; border-radius: 50%; display: flex; align-items: center; justify-content: center;
      width: 44px; height: 44px; transition: background-color 0.2s ease, transform 0.2s ease;
    }
    .icon-btn:hover { background-color: rgba(255, 255, 255, 0.1); transform: scale(1.1); }
    #play-pause-btn { font-size: 2rem; width: 64px; height: 64px; background-color: rgba(0, 0, 0, 0.3); backdrop-filter: blur(4px); }
    .timeline-container { width: 100%; cursor: pointer; padding: 0.5rem 0; }
    .timeline { height: 5px; background-color: rgba(255, 255, 255, 0.2); border-radius: 10px; position: relative; transition: height 0.2s ease; }
    .timeline-container:hover .timeline { height: 8px; }
    .progress-bar { height: 100%; width: 0; background-color: var(--tg-theme-button-color); border-radius: 10px; position: relative; transition: width 0.1s linear; }
    .timeline-container:hover .progress-bar::after {
        content: ''; position: absolute; right: -6px; top: 50%; transform: translateY(-50%);
        width: 16px; height: 16px; background-color: var(--tg-theme-button-color); border-radius: 50%; border: 2px solid var(--tg-theme-text-color);
    }
    .controls-actions { display: flex; justify-content: space-between; align-items: center; }
    .controls-left, .controls-right { display: flex; align-items: center; gap: 0.75rem; }
    .time-display { font-size: 0.875rem; font-variant-numeric: tabular-nums; color: var(--tg-theme-text-color); }
    .volume-container { display: flex; align-items: center; gap: 0.5rem; }
    input[type="range"] { -webkit-appearance: none; appearance: none; width: 80px; background: transparent; cursor: pointer; }
    input[type="range"]::-webkit-slider-runnable-track { background: rgba(255, 255, 255, 0.2); height: 5px; border-radius: 10px; }
    input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; margin-top: -5px; height: 15px; width: 15px; background: var(--tg-theme-text-color); border-radius: 50%; }
    
    .quality-menu {
        position: absolute; bottom: 70px; right: 1.25rem; background-color: rgba(31, 41, 55, 0.85);
        backdrop-filter: blur(8px); border-radius: 8px; overflow: hidden; display: none;
        flex-direction: column; z-index: 10; min-width: 100px;
    }
    .quality-menu button {
        background: none; border: none; color: var(--tg-theme-text-color); padding: 0.75rem 1.5rem;
        cursor: pointer; text-align: left; width: 100%; transition: background-color 0.2s ease;
    }
    .quality-menu button:hover { background-color: rgba(255, 255, 255, 0.1); }
    .quality-menu button.active { color: var(--tg-theme-button-color); font-weight: 600; }

    .content-section { display: flex; flex-direction: column; gap: 1rem; padding: 1rem; flex-grow: 1; overflow-y: auto; }
    .tabs {
      display: flex; background-color: var(--tg-theme-secondary-bg-color); border-radius: 10px; padding: 0.25rem; flex-shrink: 0;
      position: sticky; top: 0; z-index: 1;
    }
    .tabs button {
      flex: 1; background: none; border: none; color: var(--tg-theme-hint-color); font-weight: 600;
      font-size: 0.9rem; padding: 0.6rem 0.5rem; cursor: pointer; border-radius: 8px;
      transition: background-color 0.2s ease, color 0.2s ease;
    }
    .tabs button.active { color: var(--tg-theme-text-color); background-color: rgba(255, 255, 255, 0.1); }
    
    .playlist-grid { display: grid; gap: 1rem; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); }
    .playlist-item { cursor: pointer; transition: transform 0.2s ease; }
    .playlist-item:hover { transform: scale(1.05); }
    .playlist-item.playing .playlist-thumbnail { box-shadow: 0 0 0 3px var(--tg-theme-button-color); }
    .playlist-thumbnail { width: 100%; background-color: var(--tg-theme-secondary-bg-color); border-radius: 12px; object-fit: cover; }
    .playlist-info { margin-top: 0.5rem; }
    .playlist-title { font-weight: 600; font-size: 0.9rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .playlist-desc { font-size: 0.8rem; color: var(--tg-theme-hint-color); }
    .playlist-grid.poster-view { grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); }
    .poster-view .playlist-thumbnail { aspect-ratio: 16 / 9; }
    .playlist-grid.icon-view { grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); }
    .icon-view .playlist-item { text-align: center; }
    .icon-view .playlist-thumbnail { aspect-ratio: 1 / 1; }
    .icon-view .playlist-title { font-size: 1rem; }

    .tv-channels-container { display: flex; overflow-x: auto; padding-bottom: 1rem; gap: 1rem; scrollbar-width: none; }
    .tv-channels-container::-webkit-scrollbar { display: none; }
    .tv-channel-item { flex: 0 0 auto; width: 100px; text-align: center; cursor: pointer; transition: transform 0.2s ease; }
    .tv-channel-item:hover { transform: scale(1.05); }
    .tv-channel-thumbnail { width: 100%; aspect-ratio: 1 / 1; border-radius: 24px; object-fit: cover; background-color: var(--tg-theme-secondary-bg-color); margin-bottom: 0.5rem; }
    .tv-channel-item.playing .tv-channel-thumbnail { box-shadow: 0 0 0 3px var(--tg-theme-button-color); }
    .tv-channel-title { font-weight: 600; font-size: 1rem; }

    .native-ad-placeholder { text-align: center; padding: 2rem 1rem; background-color: var(--tg-theme-secondary-bg-color); border-radius: 12px; margin-top: 1rem; display: none; }
    .native-ad-placeholder.visible { display: block; }
  </style>
</head>
<body>

  <div class="main-container">
    <section id="player-section">
        <div class="video-player-wrapper">
            <video id="main-video" playsinline></video>
            <div class="controls-overlay">
                <div class="center-controls">
                  <button class="icon-btn" id="play-pause-btn" title="Воспроизвести/Пауза"><i class="fa-solid fa-play"></i></button>
                </div>
                <div class="controls-bottom">
                  <div class="timeline-container">
                      <div class="timeline"><div class="progress-bar"></div></div>
                  </div>
                  <div class="controls-actions">
                      <div class="controls-left">
                        <span class="time-display"><span id="current-time">00:00</span> / <span id="total-time">00:00</span></span>
                      </div>
                      <div class="controls-right">
                        <div class="volume-container">
                            <button class="icon-btn" id="mute-btn" title="Вкл/Выкл звук"><i class="fa-solid fa-volume-high"></i></button>
                            <input type="range" id="volume-slider" min="0" max="1" step="0.05" value="1">
                        </div>
                        <button class="icon-btn" id="quality-btn" title="Качество"><i class="fa-solid fa-gear"></i></button>
                        <button class="icon-btn" id="share-btn" title="Поделиться"><i class="fa-solid fa-share"></i></button>
                      </div>
                  </div>
                </div>
            </div>
            <div class="quality-menu"></div>
        </div>
    </section>

    <section class="content-section">
        <div class="tabs">
            <button id="tab-recommendations" class="active">Рекомендации</button>
            <button id="tab-sport">Спорт</button>
            <button id="tab-entertainment">Развлечения</button>
            <button id="tab-tv">ТВ</button>
        </div>
        <div id="content-display"></div>
    </section>
  </div>

  <script>
    const tg = window.Telegram.WebApp;
    try {
        tg.ready();
        tg.expand();
    } catch (e) {
        console.error("Telegram WebApp is not available.");
    }

    document.addEventListener('DOMContentLoaded', () => {
        const videoDatabase = {
            recommendations: [
                { id: "rec001", title: "Qasam", desc: "Дата и просмотры", poster: "https://i.ibb.co/GWBw1Nq/qasam-poster.jpg", sources: { '1080p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4", '720p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerFun.mp4" } },
                { id: "rec002", title: "Воин", desc: "Фэнтези, боевик", poster: "https://i.ibb.co/mX7D4C8/warrior-poster.jpg", sources: { '720p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4", '480p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4" } }
            ],
            sport: [
                { id: "sp001", title: "Futbol TV", desc: "Прямой эфир", poster: "https://i.ibb.co/3Y8219K/futbol-tv.png", sources: { 'Live': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerFun.mp4" } },
            ],
            entertainment: [
                { id: "ent001", title: "Парад машин", desc: "Зрелищное шоу", poster: "https://i.ytimg.com/vi/z18z34ciz-M/maxresdefault.jpg", sources: { '1080p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/WeAreGoingOnBullrun.mp4", '720p': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerJoyrides.mp4" } },
            ],
            tv: [
                { id: "tv001", title: "Sevimli TV", poster: "https://i.ibb.co/yQWd5dD/zor-tv.png", sources: { 'Live': "https://rt-glb.streama.com/1/hls/1-1/live.m3u8" } },
                { id: "tv002", title: "Mening Yurtim 5", poster: "https://i.ibb.co/N1gXG1g/my5-tv.png", sources: { 'Live': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerJoyrides.mp4" } },
                { id: "tv003", title: "Futbol TV", poster: "https://i.ibb.co/3Y8219K/futbol-tv.png", sources: { 'Live': "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerFun.mp4" } },
            ]
        };

        const video = document.getElementById('main-video');
        const playerWrapper = document.querySelector('.video-player-wrapper');
        const playPauseBtn = document.getElementById('play-pause-btn');
        const timelineContainer = document.querySelector('.timeline-container');
        const progressBar = document.querySelector('.progress-bar');
        const currentTimeEl = document.getElementById('current-time');
        const totalTimeEl = document.getElementById('total-time');
        const volumeSlider = document.getElementById('volume-slider');
        const muteBtn = document.getElementById('mute-btn');
        const qualityBtn = document.getElementById('quality-btn');
        const qualityMenu = document.querySelector('.quality-menu');
        const shareBtn = document.getElementById('share-btn'); // ✨ Получаем кнопку "Поделиться"
        const contentDisplay = document.getElementById('content-display');
        
        let currentVideoData = {};
        let currentCategory = 'recommendations';
        let isScrubbing = false;

        const togglePlay = () => video.paused ? video.play() : video.pause();
        video.addEventListener('play', () => { playPauseBtn.querySelector('i').className = 'fa-solid fa-pause'; playerWrapper.classList.remove('paused-state'); });
        video.addEventListener('pause', () => { playPauseBtn.querySelector('i').className = 'fa-solid fa-play'; playerWrapper.classList.add('paused-state'); });
        playPauseBtn.addEventListener('click', togglePlay);
        video.addEventListener('click', (e) => { if (e.target === video) togglePlay(); });
        video.addEventListener('timeupdate', () => { if (isScrubbing) return; const progress = (video.currentTime / video.duration) * 100; progressBar.style.width = `${progress}%`; currentTimeEl.textContent = formatTime(video.currentTime); });
        video.addEventListener('loadedmetadata', () => { totalTimeEl.textContent = formatTime(video.duration); });
        const formatTime = (seconds) => { if (isNaN(seconds)) return '00:00'; const min = Math.floor(seconds / 60); const sec = Math.floor(seconds % 60); return `${String(min).padStart(2, '0')}:${String(sec).padStart(2, '0')}`; };
        const handleTimelineUpdate = (e) => { const rect = timelineContainer.getBoundingClientRect(); const clientX = e.clientX || e.touches[0].clientX; const percent = Math.min(Math.max(0, clientX - rect.left), rect.width) / rect.width; video.currentTime = percent * video.duration; };
        timelineContainer.addEventListener('mousedown', (e) => { isScrubbing = true; handleTimelineUpdate(e); });
        document.addEventListener('mousemove', (e) => { if (isScrubbing) handleTimelineUpdate(e); });
        document.addEventListener('mouseup', () => { isScrubbing = false; });
        const handleVolumeChange = () => { video.volume = volumeSlider.value; video.muted = volumeSlider.value == 0; };
        const toggleMute = () => { video.muted = !video.muted; if(video.muted) { volumeSlider.dataset.lastValue = volumeSlider.value; volumeSlider.value = 0; } else { volumeSlider.value = volumeSlider.dataset.lastValue > 0 ? volumeSlider.dataset.lastValue : 1; } handleVolumeChange(); };
        video.addEventListener('volumechange', () => { const icon = muteBtn.querySelector('i'); if (video.muted || video.volume === 0) { icon.className = 'fa-solid fa-volume-xmark'; } else if (video.volume < 0.5) { icon.className = 'fa-solid fa-volume-low'; } else { icon.className = 'fa-solid fa-volume-high'; }});
        volumeSlider.addEventListener('input', handleVolumeChange);
        muteBtn.addEventListener('click', toggleMute);

        function switchVideo(videoData, quality = 'auto') {
            try { tg.HapticFeedback.selectionChanged(); } catch(e) {}
            currentVideoData = videoData;
            const qualities = Object.keys(videoData.sources);
            let activeQualityKey = quality === 'auto' || !videoData.sources[quality] ? qualities[qualities.length - 1] : quality;
            const sourceUrl = videoData.sources[activeQualityKey];
            const currentTime = video.currentTime;
            const isPaused = video.paused;
            video.src = sourceUrl;
            video.poster = videoData.poster;
            video.load();
            video.addEventListener('loadedmetadata', () => { video.currentTime = currentTime; if (!isPaused) video.play(); }, { once: true });
            document.querySelector('.playlist-item.playing, .tv-channel-item.playing')?.classList.remove('playing');
            const currentItem = Array.from(document.querySelectorAll('.playlist-item, .tv-channel-item')).find(item => item.dataset.id === videoData.id);
            if (currentItem) currentItem.classList.add('playing');
            renderQualityMenu(qualities, activeQualityKey);
        }

        function renderQualityMenu(qualities, activeQuality) {
            qualityMenu.innerHTML = '';
            if (qualities.length > 1) {
                qualityBtn.style.display = 'flex';
                qualities.forEach(q => {
                    const button = document.createElement('button');
                    button.textContent = q;
                    if (q === activeQuality) button.classList.add('active');
                    button.addEventListener('click', () => { switchVideo(currentVideoData, q); qualityMenu.style.display = 'none'; });
                    qualityMenu.appendChild(button);
                });
            } else {
                qualityBtn.style.display = 'none';
            }
        }
        qualityBtn.addEventListener('click', (e) => { e.stopPropagation(); qualityMenu.style.display = qualityMenu.style.display === 'flex' ? 'none' : 'flex'; });
        document.addEventListener('click', (e) => { if (!qualityMenu.contains(e.target) && e.target !== qualityBtn) qualityMenu.style.display = 'none'; });

        function renderContent(category) {
            currentCategory = category;
            const videos = videoDatabase[category];
            contentDisplay.innerHTML = '';

            if (category === 'tv') {
                const channelsContainer = document.createElement('div');
                channelsContainer.className = 'tv-channels-container';
                videos.forEach(videoInfo => {
                    const item = document.createElement('div');
                    item.className = 'tv-channel-item';
                    item.dataset.id = videoInfo.id;
                    item.innerHTML = `<img src="${videoInfo.poster}" alt="${videoInfo.title}" class="tv-channel-thumbnail"><div class="tv-channel-title">${videoInfo.title}</div>`;
                    item.addEventListener('click', () => switchVideo(videoInfo));
                    channelsContainer.appendChild(item);
                });
                contentDisplay.appendChild(channelsContainer);
                const ad = document.createElement('div');
                ad.className = 'native-ad-placeholder visible';
                ad.textContent = 'Тут нативная реклама';
                contentDisplay.appendChild(ad);
            } else {
                const gridContainer = document.createElement('div');
                gridContainer.className = (category === 'sport') ? 'playlist-grid icon-view' : 'playlist-grid poster-view';
                videos.forEach(videoInfo => {
                    const item = document.createElement('div');
                    item.className = 'playlist-item';
                    item.dataset.id = videoInfo.id;
                    item.innerHTML = `<img src="${videoInfo.poster}" alt="${videoInfo.title}" class="playlist-thumbnail"><div class="playlist-info"><div class="playlist-title">${videoInfo.title}</div>${videoInfo.desc ? `<div class="playlist-desc">${videoInfo.desc}</div>` : ''}</div>`;
                    item.addEventListener('click', () => switchVideo(videoInfo));
                    gridContainer.appendChild(item);
                });
                contentDisplay.appendChild(gridContainer);
            }
        }

        document.querySelectorAll('.tabs button').forEach(button => {
            button.addEventListener('click', () => {
                try { tg.HapticFeedback.selectionChanged(); } catch(e) {}
                document.querySelector('.tabs button.active').classList.remove('active');
                button.classList.add('active');
                renderContent(button.id.replace('tab-', ''));
            });
        });

        // ✨ 2. ЛОГИКА "ПОДЕЛИТЬСЯ" ✨
        shareBtn.addEventListener('click', () => {
            try {
                tg.HapticFeedback.notificationOccurred('success');
                // Формируем параметр для ссылки: category-videoId
                const shareParam = `${currentCategory}_${currentVideoData.id}`;
                // Открываем окно выбора чата для отправки ссылки
                tg.switchInlineQuery(shareParam, ['users', 'groups', 'channels']);
            } catch (e) {
                alert('Функция "Поделиться" доступна только в приложении Telegram.');
            }
        });

        // ✨ 3. ОБРАБОТКА ВХОДЯЩЕЙ ССЫЛКИ (DEEP LINKING) ✨
        function handleStartParam() {
            try {
                const startParam = tg.initDataUnsafe.start_param;
                if (startParam) {
                    const [category, videoId] = startParam.split('_');
                    if (videoDatabase[category]) {
                        const videoToPlay = videoDatabase[category].find(v => v.id === videoId);
                        if (videoToPlay) {
                            // Переключаем на нужную вкладку
                            document.querySelector('.tabs button.active').classList.remove('active');
                            document.getElementById(`tab-${category}`).classList.add('active');
                            // Рендерим нужный плейлист и запускаем видео
                            renderContent(category);
                            switchVideo(videoToPlay);
                            return true; // Сообщаем, что параметр обработан
                        }
                    }
                }
            } catch (e) {
                console.error("Could not parse start_param", e);
            }
            return false; // Параметр не обработан
        }

        // Инициализация
        if (!handleStartParam()) {
            // Если не было параметра для запуска, просто рендерим первую вкладку
            renderContent('recommendations');
            if (videoDatabase.recommendations.length > 0) {
                switchVideo(videoDatabase.recommendations[0]);
                video.pause();
            }
        }
    });
  </script>
</body>
</html>
