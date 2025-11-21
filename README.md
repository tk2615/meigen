<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unique High-Low Wisdom</title>
    <style>
        /* --- 重厚な賢者の書斎 --- */
        body {
            background-color: #fdfdfd;
            color: #111;
            font-family: "Yu Mincho", "YuMincho", "Hiragino Mincho ProN", serif;
            margin: 0;
            padding: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            cursor: default;
            user-select: none;
            /* 紙の質感 */
            background-image: linear-gradient(to right, rgba(0,0,0,0.03) 1px, transparent 1px), linear-gradient(to bottom, rgba(0,0,0,0.03) 1px, transparent 1px);
            background-size: 20px 20px;
        }

        /* コンテナ */
        .container {
            width: 85%;
            max-width: 900px;
            text-align: center;
            opacity: 0; 
            transition: opacity 1.0s ease-in-out;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .container.visible {
            opacity: 1;
        }

        .container.fade-out {
            opacity: 0;
            transition: opacity 1.5s ease;
        }

        /* 名言: 縦書き風の重み */
        .quote-box {
            border-top: 2px solid #111;
            border-bottom: 2px solid #111;
            padding: 40px 0;
            margin-bottom: 60px;
            width: 100%;
        }

        .quote-text {
            font-size: 2.8rem;
            font-weight: 900;
            line-height: 1.6;
            word-break: keep-all;
            letter-spacing: 0.05em;
            padding: 0 20px;
            text-shadow: 2px 2px 0 rgba(0,0,0,0.05);
        }

        /* プロフィールエリア */
        .profile-wrapper {
            display: flex;
            align-items: center;
            gap: 25px;
            justify-content: center;
            width: 100%;
        }

        /* 画像 */
        .face-img {
            width: 110px;
            height: 110px;
            border-radius: 50%;
            object-fit: cover;
            transition: filter 0.5s ease;
            border: 4px solid #fff;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        /* 名前情報 */
        .author-info {
            text-align: left;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        .author-name {
            font-size: 1.6rem;
            font-weight: bold;
            margin-bottom: 6px;
            line-height: 1.2;
            letter-spacing: 0.1em;
        }

        /* 肩書き・賞エリア */
        .author-meta {
            font-size: 0.9rem;
            color: #444;
            font-weight: normal;
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            align-items: center;
            font-family: sans-serif;
        }

        .award-text {
            color: #b8860b; /* 金色 */
            font-weight: bold;
            border-bottom: 1px solid #b8860b;
        }

        .meta-separator {
            color: #ccc;
            font-size: 0.7rem;
        }

        /* ローディング */
        .loader {
            position: absolute;
            bottom: 40px;
            font-size: 0.8rem;
            color: #888;
            letter-spacing: 0.3em;
            opacity: 0;
            transition: opacity 0.3s;
        }
        .loader.show {
            opacity: 1;
        }

        @media (max-width: 600px) {
            .quote-text { font-size: 1.8rem; }
            .profile-wrapper { flex-direction: column; gap: 20px; }
            .author-info { text-align: center; align-items: center; }
            .author-meta { justify-content: center; }
            .quote-box { padding: 20px 0; border: none; }
        }
    </style>
</head>
<body>

    <div class="container" id="main-container">
        <div class="quote-box">
            <div class="quote-text" id="quote-text"></div>
        </div>

        <div class="profile-wrapper">
            <img src="" alt="" class="face-img" id="face-img">
            <div class="author-info">
                <div class="author-name" id="author-name"></div>
                <div class="author-meta">
                    <span id="author-title"></span>
                    <span id="meta-separator" class="meta-separator" style="display:none;">/</span>
                    <span id="author-award" class="award-text"></span>
                </div>
            </div>
        </div>
    </div>

    <div class="loader" id="loader">迷言を検索中...</div>

    <script>
        /* --- 設定 --- */
        const DISPLAY_TIME = 9000;
        const FADE_TIME = 1500;
        const WIKI_API = "https://ja.wikipedia.org/w/api.php";

        /* --- DOM --- */
        const container = document.getElementById('main-container');
        const loader = document.getElementById('loader');
        const quoteEl = document.getElementById('quote-text');
        const authorNameEl = document.getElementById('author-name');
        const authorTitleEl = document.getElementById('author-title');
        const authorAwardEl = document.getElementById('author-award');
        const metaSepEl = document.getElementById('meta-separator');
        const faceImgEl = document.getElementById('face-img');

        /* --- ★履歴管理（重複完全排除）★ --- */
        // 過去に出た言葉をすべて記憶しておく
        const History = {
            quotes: new Set(),
            authors: new Set(),
            jobs: new Set(),
            awards: new Set(),
            faces: new Set()
        };

        /* --- フィルターバリエーション --- */
        const FACE_FILTERS = [
            "grayscale(100%) contrast(1.2)", 
            "sepia(0.8) contrast(1.1) brightness(0.9)",
            "contrast(1.3) saturate(0)",
            "hue-rotate(180deg) grayscale(50%)"
        ];

        /* --- 検索クエリA：高尚（フリ） --- */
        const HIGH_QUERIES = [
            "存在論", "形而上学", "量子力学", "相対性理論", "パラドックス", "ジレンマ", 
            "イデア", "無意識", "シンギュラリティ", "現象学", "実存主義", "ニヒリズム", 
            "構造主義", "多元宇宙", "暗黒物質", "エントロピー", "クオリア", "シナプス", 
            "ゲノム", "フラクタル", "輪廻", "解脱", "運命", "永遠", "カオス", "真理",
            "美学", "倫理", "正義", "国家", "革命", "文明", "歴史", "神学"
        ];

        /* --- 検索クエリB：低俗（オチ） --- */
        const LOW_QUERIES = [
            "二度寝", "夜食", "ポテチ", "ラーメン", "カフェイン", "ソシャゲ", "ガチャ", 
            "爆死", "課金", "リセマラ", "唐揚げ", "マヨネーズ", "サボり", "仮病", 
            "筋肉", "プロテイン", "サウナ", "推し", "尊い", "充電切れ", "既読スルー", 
            "黒歴史", "自分語り", "布団", "コタツ", "締め切り", "現実逃避", "積読", 
            "筋肉痛", "二日酔い", "半額", "クーポン", "借金", "留年", "便秘", "下痢",
            "マウント", "アンチ", "炎上", "論破", "遅刻", "残業", "有給"
        ];

        /* --- 検索クエリC：肩書き --- */
        const JOB_SEEDS = ["師", "家", "長", "手", "員", "王", "神", "官", "将", "隊", "係", "座", "犯", "伯", "公", "民", "マニア", "オタク"];

        /* --- フィルター --- */
        const BAD_WORDS = ["一覧", "カテゴリ", "年", "月", "日", "県", "市", "区", "町", "村", "駅", "線", "大学", "協会", "学会", "法", "製", "賞", "戦", "史", "登場人物", "放送", "アルバム", "作品"];
        const UNSAFE_WORDS = ["犯", "罪", "奴", "隷", "暴", "虐", "テロ", "襲", "撃", "狂", "痴", "春", "穢", "差別", "蔑", "詐", "欺", "刑", "囚", "殺", "死"];

        /* --- API関数 --- */
        async function fetchRandomPages() {
            const params = new URLSearchParams({ action: 'query', format: 'json', list: 'random', rnnamespace: 0, rnlimit: 40, origin: '*' });
            try { const res = await fetch(`${WIKI_API}?${params}`); const data = await res.json(); return data.query.random; } catch(e){ return []; }
        }
        async function searchTitles(queryStr, limit=20) {
            const params = new URLSearchParams({ action: 'query', format: 'json', list: 'search', srsearch: `intitle:${queryStr}`, srlimit: limit, srnamespace: 0, srsort: 'random', origin: '*' });
            try { const res = await fetch(`${WIKI_API}?${params}`); const data = await res.json(); return data.query.search.map(p => p.title); } catch (e) { return []; }
        }
        async function searchSnippets(queryStr) {
            const params = new URLSearchParams({ action: 'query', format: 'json', list: 'search', srsearch: `${queryStr}`, srlimit: 40, srnamespace: 0, origin: '*' });
            try { const res = await fetch(`${WIKI_API}?${params}`); const data = await res.json(); return data.query.search.map(p => p.snippet); } catch (e) { return []; }
        }

        /* --- 判定ロジック --- */
        function isCleanWord(text) {
            for (let bad of BAD_WORDS) { if (text.includes(bad)) return false; }
            for (let bad of UNSAFE_WORDS) { if (text.includes(bad)) return false; }
            return true;
        }
        function isSafeJob(title) {
            for (let badChar of UNSAFE_WORDS) { if (title.includes(badChar)) return false; }
            if (title.includes("一覧") || title.includes("カテゴリ")) return false; 
            return true;
        }

        /* --- 生成ロジック --- */

        function makeGlobalName(titles) {
            const validTitles = titles.filter(t => isCleanWord(t));
            const kanjiList = validTitles.filter(t => /^[一-龠]{2,}$/.test(t));
            const kanaList = validTitles.filter(t => /^[ァ-ヶー]+・[ァ-ヶー]+$/.test(t)); 
            
            let candidate = "";
            const r = Math.random();
            
            if (r < 0.5 && kanaList.length >= 2) {
                const fn = kanaList[Math.floor(Math.random()*kanaList.length)].split('・')[0];
                const ln = kanaList[Math.floor(Math.random()*kanaList.length)].split('・').pop();
                candidate = fn + "・" + ln;
            } else if (kanjiList.length >= 2) {
                const t1 = kanjiList[Math.floor(Math.random()*kanjiList.length)]; 
                const t2 = kanjiList[Math.floor(Math.random()*kanjiList.length)];
                candidate = t1 + " " + t2;
            } else {
                candidate = "ナナシ・ゴンベエ";
            }
            return candidate;
        }

        // 高低差コラージュ生成
        function mixHighLow(highSnippets, lowSnippets) {
            let upperParts = [];
            let lowerParts = [];

            // 上の句（高尚）: 「〜とは、」「〜において、」「〜という概念は、」
            const highRegex = /([^、。！？\s]{4,20}(とは|においては|という概念は|の本質は|の真理は))、/g;
            
            // 下の句（低俗）: 「〜である。」「〜してしまう。」「〜に過ぎない。」「〜最強。」
            const lowRegex = /、([^、。！？\s]{5,20}(である|だ|してしまう|した|ない|最強|マシ|草|不可避|詰んだ|運命|バグ|真実|限界|罪))([。！？])/g;

            highSnippets.forEach(snip => {
                let text = snip.replace(/<[^>]*>/g, "").replace(/\[.*?\]/g, "");
                const matches = [...text.matchAll(highRegex)];
                matches.forEach(m => upperParts.push(m[1] + "、"));
            });

            lowSnippets.forEach(snip => {
                let text = snip.replace(/<[^>]*>/g, "").replace(/\[.*?\]/g, "");
                const matches = [...text.matchAll(lowRegex)];
                matches.forEach(m => lowerParts.push(m[1] + m[3]));
            });

            if (upperParts.length > 0 && lowerParts.length > 0) {
                // 試行して重複しない組み合わせを探す
                for(let i=0; i<20; i++) {
                    const upper = upperParts[Math.floor(Math.random() * upperParts.length)];
                    const lower = lowerParts[Math.floor(Math.random() * lowerParts.length)];
                    const cleanUpper = upper.replace(/^[、。！？]/, "").replace(/^[^一-龠ァ-ヶーa-zA-Z0-9]+/, "");
                    const combined = cleanUpper + lower;

                    // ★履歴チェック★
                    if (combined.length <= 50 && !History.quotes.has(combined) && !/[（）\(\)『』「」]/.test(combined)) {
                        return combined;
                    }
                }
            }
            return null;
        }

        /* --- メインサイクル --- */
        async function startCycle() {
            container.classList.remove('visible');
            if (quoteEl.textContent !== "") {
                container.classList.add('fade-out');
            }
            
            loader.classList.add('show');
            await new Promise(r => setTimeout(r, FADE_TIME));

            container.classList.remove('fade-out');
            authorAwardEl.textContent = "";
            metaSepEl.style.display = "none";

            let quote = null, author = null, job = null, award = null;
            let retry = 0;

            while((!quote || !author || !job) && retry < 30) { 
                try {
                    const pRandom = fetchRandomPages(); 
                    
                    // 1. 高尚なフリ
                    const highQ = HIGH_QUERIES[Math.floor(Math.random() * HIGH_QUERIES.length)];
                    const pHighSnippets = searchSnippets(highQ);

                    // 2. 低俗なオチ
                    const lowQ = LOW_QUERIES[Math.floor(Math.random() * LOW_QUERIES.length)];
                    const pLowSnippets = searchSnippets(lowQ);

                    // 3. 肩書き
                    const jobSeed = JOB_SEEDS[Math.floor(Math.random() * JOB_SEEDS.length)];
                    const pJobTitles = searchTitles(jobSeed, 20);

                    // 4. 賞
                    const fetchAward = Math.random() < 0.3;
                    const pAwards = fetchAward ? searchTitles("賞", 10) : Promise.resolve([]);

                    const [randPages, highSnips, lowSnips, jobTitles, awardTitles] = await Promise.all([pRandom, pHighSnippets, pLowSnippets, pJobTitles, pAwards]);

                    // --- 名言生成 ---
                    if (highSnips.length > 0 && lowSnips.length > 0) {
                        quote = mixHighLow(highSnips, lowSnips);
                    }

                    // --- 名前生成 ---
                    // 重複チェックしながら生成
                    const combinedNameSrc = [...randPages.map(p=>p.title), ...jobTitles];
                    let tempAuthor = makeGlobalName(combinedNameSrc);
                    if (!History.authors.has(tempAuthor)) {
                        author = tempAuthor;
                    }

                    // --- 肩書き生成 ---
                    const validJobs = jobTitles.filter(t => { 
                        return t.endsWith(jobSeed) && isCleanWord(t) && t.length >= 2 && t.length <= 12 && isSafeJob(t) && !History.jobs.has(t); 
                    });
                    if (validJobs.length > 0) { 
                        job = validJobs[Math.floor(Math.random() * validJobs.length)]; 
                    }

                    // --- 賞 ---
                    const validAwards = awardTitles.filter(t => t.endsWith("賞") && isCleanWord(t) && !History.awards.has(t));
                    if (validAwards.length > 0) { 
                        award = validAwards[Math.floor(Math.random() * validAwards.length)]; 
                    }

                } catch(e) { console.error(e); }
                retry++;
            }

            // フォールバック
            if(!quote) quote = "存在論のパラドックスとは、二度寝である。";
            if(!author) author = "ジョン・スミス";
            if(!job) job = "哲学者";

            // ★履歴登録★
            History.quotes.add(quote);
            History.authors.add(author);
            History.jobs.add(job);
            if(award) History.awards.add(award);

            // 画像生成
            let uniqueId;
            let safety = 0;
            do { 
                uniqueId = Math.random().toString(36).substring(2, 15); 
                safety++; 
            } while (History.faces.has(uniqueId) && safety < 100); 
            History.faces.add(uniqueId);

            const faceUrl = `https://picsum.photos/seed/${uniqueId}/150/150`;
            const randomFilter = FACE_FILTERS[Math.floor(Math.random() * FACE_FILTERS.length)];

            await new Promise(r => { const i=new Image(); i.onload=r; i.onerror=r; i.src=faceUrl; });

            quoteEl.textContent = quote;
            authorNameEl.textContent = author;
            authorTitleEl.textContent = job;
            
            if (award) {
                metaSepEl.style.display = "inline";
                authorAwardEl.textContent = award;
            } else {
                metaSepEl.style.display = "none";
                authorAwardEl.textContent = "";
            }

            faceImgEl.src = faceUrl;
            faceImgEl.style.filter = randomFilter;

            loader.classList.remove('show');
            container.classList.add('visible');

            await new Promise(r => setTimeout(r, DISPLAY_TIME));
            startCycle();
        }

        window.onload = startCycle;

    </script>
</body>
</html>
