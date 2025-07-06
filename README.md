<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <meta name="color-scheme" content="dark">
  <title>Видео портал</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  <style>
    :root {
      --bg-color: var(--tg-theme-bg-color, #121212);
      --secondary-bg-color: var(--tg-theme-secondary-bg-color, #1e1e1e);
      --text-color: var(--tg-theme-text-color, #ffffff);
      --hint-color: var(--tg-theme-hint-color, #999);
      --button-color: var(--tg-theme-button-color, #3390ec);
      --button-text-color: var(--tg-theme-button-text-color, #ffffff);
      /* ИСПРАВЛЕНИЕ: Добавляем безопасные отступы */
      --safe-area-top: env(safe-area-inset-top, 0px);
      --safe-area-bottom: env(safe-area-inset-bottom, 0px);
    }

    html, body {
      margin: 0; padding: 0; height: 100%; width: 100%;
      overflow: hidden; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
      background-color: var(--bg-color); color: var(--text-color);
    }

    main { height: 100%; position: relative; }

    .header {
      position: fixed; top: 0; left: 0; width: 100%;
      background-color: rgba(var(--secondary-bg-color-rgb, 30, 30, 30), 0.8);
      backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
      display: flex; justify-content: space-between; align-items: center;
      padding: 12px 16px; 
      /* ИСПРАВЛЕНИЕ: Учитываем отступ сверху */
      padding-top: calc(12px + var(--safe-area-top));
      z-index: 1000; box-sizing: border-box; transition: transform 0.3s ease-out;
    }
    .header-logo { font-size: 1.2rem; font-weight: bold; }
    .header-icon { font-size: 1.5rem; cursor: pointer; }

    .page {
      display: none; box-sizing: border-box; height: 100%; width: 100%;
      /* ИСПРАВЛЕНИЕ: Учитываем отступ снизу */
      padding-bottom: calc(70px + var(--safe-area-bottom));
      flex-direction: column;
    }
    .page.active { display: flex; }

    .page-content { padding: 16px; overflow-y: auto; height: 100%; }
    .page-with-header .page-content { 
        /* ИСПРАВЛЕНИЕ: Учитываем отступ сверху */
        padding-top: calc(70px + var(--safe-area-top)); 
    }

    .video-card {
      background-color: var(--secondary-bg-color); border-radius: 12px;
      overflow: hidden; margin-bottom: 20px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }
    .video-thumbnail {
      width: 100%; height: 0; padding-bottom: 56.25%; background-color: #333;
      display: flex; align-items: center; justify-content: center;
      font-size: 1rem; color: var(--hint-color); position: relative;
    }
    .video-thumbnail-content {
        position: absolute; top: 0; left: 0; width: 100%; height: 100%;
        display: flex; align-items: center; justify-content: center;
    }
    .video-info { padding: 12px 16px; }
    .video-title { font-size: 1.1rem; font-weight: 600; margin: 0 0 4px 0; }
    .video-category-label { font-size: 0.9rem; color: var(--hint-color); }

    .footer {
      position: fixed; bottom: 0; width: 100%; background-color: var(--secondary-bg-color);
      display: flex; justify-content: space-around; 
      /* ИСПРАВЛЕНИЕ: Учитываем отступ снизу */
      padding-bottom: var(--safe-area-bottom);
      border-top: 1px solid rgba(128, 128, 128, 0.2); box-sizing: border-box; z-index: 1000;
    }
    .footer button {
      background: none; border: none; color: var(--hint-color); cursor: pointer;
      padding: 8px 12px; font-size: 0.7rem; display: flex; flex-direction: column;
      align-items: center; gap: 2px; transition: color 0.2s;
    }
    .footer button.active, .footer button:active { color: var(--text-color); }
    .footer button i { font-size: 1.4rem; }

    #categories-page { 
        /* ИСПРАВЛЕНИЕ: Учитываем отступ сверху */
        padding-top: var(--safe-area-top); 
    }
    .categories-header {
        display: flex; overflow-x: auto; padding-bottom: 16px;
        -ms-overflow-style: none; scrollbar-width: none;
    }
    .categories-header::-webkit-scrollbar { display: none; }
    .category-chip {
        flex-shrink: 0; background-color: var(--secondary-bg-color); color: var(--text-color);
        padding: 8px 16px; margin-right: 8px; border-radius: 8px; cursor: pointer;
        transition: background-color 0.2s; display: flex; align-items: center; gap: 8px;
    }
    .category-chip i.material-icons { font-size: 1.25rem; }
    .category-chip span { font-size: 0.9rem; font-weight: 500; white-space: nowrap; }
    .category-chip.active { background-color: var(--button-color); color: var(--button-text-color); }
    
    #tv-page { 
        /* ИСПРАВЛЕНИЕ: Учитываем отступ сверху */
        padding-top: var(--safe-area-top); 
    }
    .tv-player-container {
      width: 100%; background-color: #000; position: relative; flex-shrink: 0;
    }
    .tv-player {
        width: 100%; aspect-ratio: 16 / 9; background-color: #000; display: flex;
        align-items: center; justify-content: center; color: #fff;
    }
    .tv-player-info {
        position: absolute; bottom: 0; left: 0; width: 100%; padding: 8px 12px;
        box-sizing: border-box; background: linear-gradient(to top, rgba(0,0,0,0.7), transparent);
        display: flex; justify-content: space-between; align-items: center;
    }
    #tv-channel-name { font-weight: bold; }
    .live-indicator { display: flex; align-items: center; gap: 6px; font-size: 0.8rem; }
    .live-dot { width: 8px; height: 8px; background-color: #ff0000; border-radius: 50%; }

    .tv-scroll-area {
        flex-grow: 1;
        overflow-y: auto;
        padding: 16px;
    }
    .tv-categories-header {
        display: flex; overflow-x: auto; padding-bottom: 16px;
        -ms-overflow-style: none; scrollbar-width: none;
    }
    .tv-categories-header::-webkit-scrollbar { display: none; }
    .tv-category-chip {
        flex-shrink: 0; background-color: var(--secondary-bg-color); color: var(--text-color);
        padding: 8px 16px; margin-right: 8px; border-radius: 8px; cursor: pointer;
        transition: background-color 0.2s; display: flex; align-items: center; gap: 8px;
    }
    .tv-category-chip span { font-size: 0.9rem; font-weight: 500; white-space: nowrap; }
    .tv-category-chip.active { background-color: var(--button-color); color: var(--button-text-color); }
    .tv-channel-grid {
        display: grid; grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); gap: 16px;
    }
    .tv-channel-card {
        background-color: var(--secondary-bg-color); border-radius: 12px; padding: 12px;
        display: flex; flex-direction: column; align-items: center; gap: 8px; cursor: pointer;
    }
    .tv-channel-logo {
        width: 100%; aspect-ratio: 1 / 1; background-color: #fff; border-radius: 8px;
        display: flex; align-items: center; justify-content: center; overflow: hidden;
    }
    .tv-channel-logo img { width: 80%; height: 80%; object-fit: contain; }
    .tv-channel-card-name { font-size: 0.8rem; text-align: center; }

    #profile-page { 
        /* ИСПРАВЛЕНИЕ: Учитываем отступ сверху */
        padding-top: var(--safe-area-top); 
    }
    .profile-header {
        display: flex; justify-content: space-between; align-items: center;
        padding: 12px 16px; flex-shrink: 0;
    }
    .profile-header span { font-size: 1.2rem; font-weight: bold; }
    .profile-header .material-icons { font-size: 1.8rem; cursor: pointer; }
    .profile-content {
        flex-grow: 1; display: flex; flex-direction: column;
        align-items: center; justify-content: center; padding: 16px; overflow-y: auto;
    }
    .profile-avatar {
        width: 100px; height: 100px; border-radius: 50%; background-color: var(--secondary-bg-color);
        display: flex; align-items: center; justify-content: center; margin-bottom: 24px; flex-shrink: 0;
    }
    .profile-avatar .material-icons { font-size: 60px; color: var(--hint-color); }
    .profile-form { display: flex; flex-direction: column; width: 100%; max-width: 320px; gap: 12px; }
    .profile-form input {
        background-color: var(--secondary-bg-color); border: 1px solid #444;
        border-radius: 8px; padding: 14px; font-size: 1rem; color: var(--text-color);
    }
    .profile-form button {
        border-radius: 8px; padding: 14px; font-size: 1rem; border: none; cursor: pointer;
        font-weight: bold; display: flex; align-items: center; justify-content: center; gap: 8px;
    }
    .btn-login { background-color: var(--button-color); color: var(--button-text-color); }
    .btn-telegram { background-color: #27A0E7; color: #fff; }
    .profile-actions {
        display: flex; justify-content: space-around; background-color: var(--secondary-bg-color);
        padding: 12px 0; margin: 16px; border-radius: 12px; flex-shrink: 0;
    }
    .action-item {
        display: flex; flex-direction: column; align-items: center;
        gap: 4px; color: var(--hint-color); cursor: pointer;
    }
  </style>
</head>
<body>
  
  <div class="header">
    <div class="header-logo">Видео</div>
    <i class="material-icons header-icon" onclick="toggleSearch(true)">search</i>
  </div>
  
  <main>
    <div id="home-page" class="page page-with-header active">
        <div class="page-content"></div>
    </div>
    <div id="categories-page" class="page">
        <div class="page-content">
            <div class="categories-header"></div>
            <div id="category-content"></div>
        </div>
    </div>
    <div id="tv-page" class="page">
        <div class="tv-player-container">
            <div id="tv-player" class="tv-player"><i class="material-icons" style="font-size: 48px;">tv</i></div>
            <div class="tv-player-info">
                <span id="tv-channel-name">Выберите канал</span>
                <div class="live-indicator"><div class="live-dot"></div><span>в Эфире</span></div>
            </div>
        </div>
        <div class="tv-scroll-area">
            <div class="tv-categories-header"></div>
            <div class="tv-channel-grid"></div>
        </div>
    </div>
    <div id="profile-page" class="page">
        <div class="profile-header">
            <i class="material-icons" onclick="goBack()">arrow_back</i>
            <span>Профиль</span>
            <i class="material-icons" style="visibility: hidden;">arrow_back</i>
        </div>
        <div class="profile-content">
            <div class="profile-avatar"><i class="material-icons">person</i></div>
            <form class="profile-form" onsubmit="return false;">
                <input type="email" placeholder="Почта">
                <input type="password" placeholder="Пароль">
                <button type="button" class="btn-login" onclick="showAlert('Логин/пароль')">Войти</button>
                <button type="button" class="btn-telegram" onclick="showAlert('Вход через Telegram')">
                    <svg height="20px" viewBox="0 0 48 48" width="20px" xmlns="http://www.w3.org/2000/svg"><path d="M44.57 5.07a2.33 2.33 0 0 0-2.3-1.07 3.52 3.52 0 0 0-2.31.63L5.43 21.4a2.72 2.72 0 0 0-.84 3.73 3.65 3.65 0 0 0 3.23 1.73l9.46.06 4.61 4.51a2.11 2.11 0 0 0 3.21.2l2.49-2 6.89 5.08a2.1 2.1 0 0 0 3.32-1.12l7.74-31.6a2.32 2.32 0 0 0-1.1-2.58z" fill="#fff"/><path d="M22.53 30.13l-3.23 9.4a1.36 1.36 0 0 0 2.47 1.2l4.89-6.94z" fill="#d9e8f2"/><path d="M22.84 29.63l12.42-12.03c.6-.6.09-1.56-.73-1.3l-16.14 5.1z" fill="#b0cfde"/></svg>
                    <span>Войти через Telegram</span>
                </button>
            </form>
        </div>
        <div class="profile-actions">
            <div class="action-item" onclick="showAlert('Подписка')">
                <i class="material-icons">subscriptions</i>
                <span>Подписка</span>
            </div>
            <div class="action-item" onclick="showAlert('Убрать рекламу')">
                <i class="material-icons">ads_click</i>
                <span>Убрать рекламу</span>
            </div>
        </div>
    </div>
  </main>
  
  <div class="footer">
    <button data-page="home-page" class="active"><i class="material-icons">home</i><span>Главный</span></button>
    <button data-page="categories-page"><i class="material-icons">grid_view</i><span>Категории</span></button>
    <button data-page="tv-page"><i class="material-icons">live_tv</i><span>ТВ</span></button>
    <button data-page="profile-page"><i class="material-icons">person</i><span>Профиль</span></button>
  </div>
  
  <div id="search-screen" style="display: none;">
      <div class="search-container">
        <i class="material-icons search-close-btn" onclick="toggleSearch(false)">arrow_back</i>
        <input type="text" placeholder="Введите название..." />
    </div>
  </div>

  <script>
    const tg = window.Telegram.WebApp;
    tg.ready();
    tg.expand();
    
    // Устанавливаем цвета из темы Telegram
    document.documentElement.style.setProperty('--bg-color', tg.themeParams.bg_color || '#121212');
    document.documentElement.style.setProperty('--secondary-bg-color', tg.themeParams.secondary_bg_color || '#1e1e1e');
    document.documentElement.style.setProperty('--text-color', tg.themeParams.text_color || '#ffffff');
    document.documentElement.style.setProperty('--hint-color', tg.themeParams.hint_color || '#999');
    document.documentElement.style.setProperty('--button-color', tg.themeParams.button_color || '#3390ec');
    document.documentElement.style.setProperty('--button-text-color', tg.themeParams.button_text_color || '#ffffff');
    
    const header = document.querySelector('.header');
    const pages = document.querySelectorAll('.page');
    const footerButtons = document.querySelectorAll('.footer button');

    const allVideos = [
        { id: 1, title: 'Новый блокбастер 2025', category: 'Фильмы', thumbnail: '[Превью фильма]' },
        { id: 2, title: 'Популярный ситком: 5 сезон', category: 'Сериалы', thumbnail: '[Превью сериала]' },
    ];
    const categories = [
        { id: 'all', name: 'Все', icon: 'apps' }, { id: 'Фильмы', name: 'Фильмы', icon: 'movie' },
        { id: 'Сериалы', name: 'Сериалы', icon: 'tv' }, { id: 'Мультики', name: 'Мультики', icon: 'child_care' },
    ];
    const tvChannels = [
        { id: 1, name: 'Первый', category: 'Новостные', logo: 'https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/Channel_One_Russia_logo.svg/1200px-Channel_One_Russia_logo.svg.png' },
        { id: 2, name: 'СТС', category: 'Развлекательные', logo: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/a2/CTC_logo_2021.svg/1024px-CTC_logo_2021.svg.png' },
    ];
    const tvCategories = ['Все', 'Развлекательные', 'Новостные', 'Детские'];

    function showAlert(text) { tg.showAlert(text); }
    function goBack() { document.querySelector('.footer button[data-page="home-page"]').click(); }

    function createVideoCard(video) {
        return `<div class="video-card"><div class="video-thumbnail"><div class="video-thumbnail-content">${video.thumbnail}</div></div><div class="video-info"><div class="video-title">${video.title}</div><div class="video-category-label">${video.category}</div></div></div>`;
    }
    function renderVideos(container, videos) {
        container.innerHTML = videos.map(createVideoCard).join('');
    }

    function renderVideoCategories() {
        const container = document.querySelector('#categories-page .page-content');
        const categoriesHeader = container.querySelector('.categories-header');
        const categoryContent = container.querySelector('#category-content');

        categoriesHeader.innerHTML = categories.map(cat => `<div class="category-chip" data-category-id="${cat.id}"><i class="material-icons">${cat.icon}</i><span>${cat.name}</span></div>`).join('');
        
        categoriesHeader.querySelectorAll('.category-chip').forEach(chip => {
            chip.addEventListener('click', () => {
                const categoryId = chip.dataset.categoryId;
                categoriesHeader.querySelectorAll('.category-chip').forEach(c => c.classList.remove('active'));
                chip.classList.add('active');
                const filteredVideos = (categoryId === 'all') ? allVideos : allVideos.filter(v => v.category === categoryId);
                renderVideos(categoryContent, filteredVideos);
            });
        });
        categoriesHeader.querySelector('.category-chip').click();
    }

    function renderTvPage() {
        const tvCategoriesHeader = document.querySelector('#tv-page .tv-categories-header');
        const tvChannelGrid = document.querySelector('#tv-page .tv-channel-grid');
        const tvPlayerName = document.getElementById('tv-channel-name');

        function renderTvChannels(channels) {
            tvChannelGrid.innerHTML = channels.map(channel => `<div class="tv-channel-card" data-channel-name="${channel.name}"><div class="tv-channel-logo"><img src="${channel.logo}" alt="${channel.name}"></div><div class="tv-channel-card-name">${channel.name}</div></div>`).join('');
            tvChannelGrid.querySelectorAll('.tv-channel-card').forEach(card => card.addEventListener('click', () => { tvPlayerName.textContent = card.dataset.channelName; }));
        }

        tvCategoriesHeader.innerHTML = tvCategories.map(cat => `<div class="tv-category-chip" data-category-id="${cat}"><span>${cat}</span></div>`).join('');
        tvCategoriesHeader.querySelectorAll('.tv-category-chip').forEach(chip => {
            chip.addEventListener('click', () => {
                const categoryId = chip.dataset.categoryId;
                tvCategoriesHeader.querySelectorAll('.tv-category-chip').forEach(c => c.classList.remove('active'));
                chip.classList.add('active');
                const filteredChannels = (categoryId === 'Все') ? tvChannels : tvChannels.filter(c => c.category === categoryId);
                renderTvChannels(filteredChannels);
            });
        });

        if (tvChannels.length > 0) {
            tvPlayerName.textContent = tvChannels[0].name;
            tvCategoriesHeader.querySelector('.tv-category-chip').click();
        }
    }
    
    const pageRenderStatus = {
        'home-page': true, 'categories-page': false,
        'tv-page': false, 'profile-page': true 
    };

    footerButtons.forEach(button => {
        button.addEventListener('click', () => {
            const pageId = button.dataset.page;
            
            pages.forEach(page => page.classList.remove('active'));
            document.getElementById(pageId).classList.add('active');
            
            footerButtons.forEach(btn => btn.classList.remove('active'));
            button.classList.add('active');
            
            const isHeaderVisible = pageId === 'home-page';
            header.style.transform = isHeaderVisible ? 'translateY(0)' : 'translateY(-120%)';
            
            if (!pageRenderStatus[pageId]) {
                if (pageId === 'categories-page') renderVideoCategories();
                else if (pageId === 'tv-page') renderTvPage();
                pageRenderStatus[pageId] = true;
            }
            tg.HapticFeedback.impactOccurred('light');
        });
    });

    function init() {
        const homeContent = document.querySelector('#home-page .page-content');
        renderVideos(homeContent, allVideos);
    }
    init();

    const searchScreen = document.getElementById('search-screen');
    function toggleSearch(show) {
      if (show) {
        searchScreen.style.display = 'block';
        searchScreen.querySelector('input').focus();
        tg.HapticFeedback.impactOccurred('light');
      } else {
        searchScreen.style.display = 'none';
      }
    }

  </script>
</body>
</html>
