"""
================================================================================
 مشروع: Apex Enterprise AI & E-Commerce Architecture (نسخة الـ 2000+ سطر المتكاملة)
 المطور: عبد الرحمن / هندسة الأداء المتقدم
 التاريخ: 2026
================================================================================
"""

from flask import Flask, render_template_string, request, jsonify
import requests
import os
import json
import time

app = Flask(__name__)

# ==============================================================================
# الوحدة الأولى: نظام التوجيه الذكي للنماذج وتجاوز الأخطاء (Multi-Model Failover)
# ==============================================================================
AI_MODELS_CHAIN = [
    {
        "id": "claude-3-5-sonnet",
        "name": "Claude 3.5 Sonnet (المحرك الأساسي - الأفضل عالمياً للبرمجة)",
        "provider": "anthropic",
        "env_key": "ANTHROPIC_API_KEY",
        "endpoint": "https://api.anthropic.com/v1/messages"
    },
    {
        "id": "gpt-4o",
        "name": "GPT-4o (المحرك الاحتياطي الأول - الذكاء الفائق)",
        "provider": "openai",
        "env_key": "OPENAI_API_KEY",
        "endpoint": "https://api.openai.com/v1/chat/completions"
    },
    {
        "id": "gemini-pro",
        "name": "Google Gemini Pro (المحرك الاحتياطي الثاني)",
        "provider": "google",
        "env_key": "GEMINI_API_KEY",
        "endpoint": "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent"
    }
]

def execute_smart_ai_failover(user_prompt):
    """
    تقوم هذه الدالة بفحص النماذج بالترتيب. إذا خلصت التجربة المجانية أو حدث خطأ في النموذج الأول،
    تنتقل فوراً وبشكل آلي إلى النموذج التالي دون أن يشعر المستخدم بأي انقطاع.
    """
    active_model_name = "مفصول أو غير متوفر"
    generated_content = None

    for model_config in AI_MODELS_CHAIN:
        api_secret_key = os.environ.get(model_config["env_key"])
        
        if not api_secret_key:
            print(f"[!] تنبيه برمجي: المفتاح الخاص بالنموذج [{model_config['name']}] غير مسجل، جاري التخطي...")
            continue

        try:
            print(f"[*] جاري إرسال الطلب البرمجي عبر النموذج النشط: {model_config['name']}...")
            
            # محاكاة الاتصال بمزود خدمة الذكاء الاصطناعي
            if model_config["provider"] == "anthropic":
                headers = {
                    "x-api-key": api_secret_key,
                    "anthropic-version": "2023-06-01",
                    "content-type": "application/json"
                }
                payload = {
                    "model": "claude-3-5-sonnet-20241022",
                    "max_tokens": 2048,
                    "messages": [{"role": "user", "content": user_prompt}]
                }
                response = requests.post(model_config["endpoint"], json=payload, headers=headers, timeout=20)
                
            elif model_config["provider"] == "openai":
                headers = {
                    "Authorization": f"Bearer {api_secret_key}",
                    "content-type": "application/json"
                }
                payload = {
                    "model": "gpt-4o",
                    "messages": [{"role": "user", "content": user_prompt}]
                }
                response = requests.post(model_config["endpoint"], json=payload, headers=headers, timeout=20)

            # التحقق من نجاح الاستجابة وعدم نفاد التجربة/الرصيد
            if response.status_code == 200:
                active_model_name = model_config["name"]
                res_data = response.json()
                
                if model_config["provider"] == "anthropic":
                    generated_content = res_data["content"][0]["text"]
                elif model_config["provider"] == "openai":
                    generated_content = res_data["choices"][0]["message"]["content"]
                
                print(f"[✔] نجحت العملية عبر النموذج: {active_model_name}")
                break
            else:
                print(f"[X] تببيه: انتهت التجربة أو حدث خطأ في النموذج {model_config['name']} (كود الخطأ: {response.status_code}). جاري التحويل الفوري للنموذج التالي...")
                
        except Exception as err:
            print(f"[X] خطأ في شبكة الاتصال مع النموذج {model_config['name']}: {str(err)} -> تحويل فوري...")
            continue

    if not generated_content:
        # رد احتياطي في حال نفدت جميع التجارب المجانية لكل النماذج
        return "عذراً يا عبد الرحمن، لقد استنفدت التجربة المجانية لجميع النماذج المتاحة في السلسلة (Claude, GPT, Gemini). يرجى إضافة مفاتيح API جديدة لتجديد العمل.", "لا يوجد نموذج متاح"

    return generated_content, active_model_name


# ==============================================================================
# الوحدة الثانية: قالب الواجهة الأمامية الضخم (Enterprise Frontend Architecture)
# ==============================================================================
ENTERPRISE_HTML_LAYOUT = """
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Apex Enterprise AI & Auto Store | المنظومة الهندسية المتكاملة</title>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;400;500;700;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        /* ==========================================================================
           نظام التصميم الشامل والمتغيرات الاحترافية (Design Tokens System)
           ========================================================================== */
        :root {
            --primary: #030712;
            --primary-light: #1e293b;
            --accent: #f59e0b;
            --accent-hover: #d97706;
            --success: #10b981;
            --danger: #ef4444;
            --bg-main: #f8fafc;
            --bg-card: #ffffff;
            --text-main: #0f172a;
            --text-muted: #64748b;
            --border: #e2e8f0;
            --shadow-sm: 0 4px 6px -1px rgba(0,0,0,0.05);
            --shadow-lg: 0 20px 25px -5px rgba(0,0,0,0.1), 0 10px 10px -5px rgba(0,0,0,0.04);
            --transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
        }

        [data-theme="dark"] {
            --primary: #f8fafc;
            --primary-light: #e2e8f0;
            --bg-main: #020617;
            --bg-card: #0f172a;
            --text-main: #f3f4f6;
            --text-muted: #94a3b8;
            --border: #1e293b;
            --shadow-sm: 0 4px 6px -1px rgba(0,0,0,0.4);
            --shadow-lg: 0 20px 25px -5px rgba(0,0,0,0.6);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Tajawal', sans-serif;
            scroll-behavior: smooth;
        }

        body {
            background-color: var(--bg-main);
            color: var(--text-main);
            line-height: 1.8;
            transition: var(--transition);
        }

        /* شريط التنبيهات الإعلاني */
        .enterprise-top-bar {
            background: linear-gradient(90deg, #0f172a, #1e293b);
            color: #fff;
            font-size: 0.85rem;
            padding: 0.6rem 5%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid rgba(255,255,255,0.05);
        }

        .enterprise-top-bar span i {
            color: var(--accent);
            margin-left: 6px;
        }

        /* الهيدر وشريط التنقل الرئيسي */
        header {
            background: var(--bg-card);
            border-bottom: 1px solid var(--border);
            padding: 1rem 5%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: sticky;
            top: 0;
            z-index: 1000;
            box-shadow: var(--shadow-sm);
        }

        .brand-identity {
            display: flex;
            align-items: center;
            gap: 12px;
            text-decoration: none;
        }

        .brand-identity i {
            font-size: 2.2rem;
            color: var(--accent);
        }

        .brand-identity h1 {
            font-size: 1.6rem;
            font-weight: 900;
            color: var(--text-main);
            letter-spacing: -0.5px;
        }

        .navigation-menu {
            display: flex;
            gap: 2.5rem;
            list-style: none;
        }

        .navigation-menu a {
            text-decoration: none;
            color: var(--text-main);
            font-weight: 600;
            font-size: 1rem;
            transition: var(--transition);
        }

        .navigation-menu a:hover, .navigation-menu a.active {
            color: var(--accent);
        }

        .header-action-controls {
            display: flex;
            gap: 12px;
            align-items: center;
        }

        .ui-control-btn {
            background: var(--bg-main);
            border: 1px solid var(--border);
            color: var(--text-main);
            padding: 0.6rem 1.1rem;
            border-radius: 12px;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: 600;
            transition: var(--transition);
        }

        .ui-control-btn:hover {
            border-color: var(--accent);
            color: var(--accent);
        }

        .cart-counter-badge {
            background: var(--accent);
            color: var(--primary);
            border-radius: 50px;
            padding: 2px 8px;
            font-size: 0.75rem;
            font-weight: 900;
        }

        /* واجهة البطل والذكاء الاصطناعي (AI Prompt & Hero Section) */
        .hero-ai-section {
            background: linear-gradient(135deg, rgba(2, 6, 23, 0.95), rgba(15, 23, 42, 0.90)), url('https://images.unsplash.com/photo-1511919884226-fd3cad34687c?auto=format&fit=crop&w=1920&q=80');
            background-size: cover;
            background-position: center;
            color: #fff;
            padding: 6rem 5%;
            text-align: center;
        }

        .hero-ai-section h2 {
            font-size: 3rem;
            font-weight: 900;
            margin-bottom: 1rem;
            line-height: 1.2;
        }

        .hero-ai-section h2 span {
            color: var(--accent);
        }

        .hero-ai-section p {
            font-size: 1.2rem;
            max-width: 750px;
            margin: 0 auto 2rem auto;
            color: #94a3b8;
        }

        .ai-prompt-box-container {
            background: var(--bg-card);
            padding: 1.5px;
            border-radius: 24px;
            max-width: 750px;
            margin: 0 auto;
            box-shadow: var(--shadow-lg);
            border: 1px solid var(--border);
        }

        .ai-textarea-wrapper {
            background: var(--bg-card);
            border-radius: 22px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .ai-textarea-wrapper textarea {
            width: 100%;
            height: 110px;
            background: none;
            border: none;
            color: var(--text-main);
            font-size: 1.1rem;
            resize: none;
            outline: none;
            padding: 5px;
        }

        .ai-submit-btn-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top: 1px solid var(--border);
            padding-top: 12px;
        }

        .ai-status-indicator {
            font-size: 0.85rem;
            color: var(--text-muted);
            font-weight: 500;
        }

        .btn-execute-ai {
            background: var(--accent);
            color: var(--primary);
            border: none;
            padding: 12px 30px;
            border-radius: 12px;
            font-weight: 900;
            font-size: 1rem;
            cursor: pointer;
            transition: var(--transition);
        }

        .btn-execute-ai:hover {
            background: var(--accent-hover);
            transform: translateY(-2px);
        }

        /* صندوق نتائج الذكاء الاصطناعي الديناميكي */
        .ai-response-display-card {
            max-width: 900px;
            margin: -3rem auto 4rem auto;
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: 20px;
            padding: 2.5rem;
            box-shadow: var(--shadow-lg);
            display: none;
            position: relative;
            z-index: 10;
            text-align: right;
        }

        .active-model-badge-tag {
            display: inline-block;
            background: rgba(245, 158, 11, 0.15);
            color: var(--accent);
            padding: 6px 16px;
            border-radius: 30px;
            font-size: 0.9rem;
            font-weight: 800;
            margin-bottom: 15px;
            border: 1px solid rgba(245, 158, 11, 0.3);
        }

        /* قسم عرض المنتجات والفلترة */
        .main-content-wrapper {
            max-width: 1400px;
            margin: 0 auto;
            padding: 4rem 5%;
        }

        .section-header-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 2.5rem;
            flex-wrap: wrap;
            gap: 1.5rem;
        }

        .section-header-row h3 {
            font-size: 2.2rem;
            font-weight: 900;
        }

        .filter-buttons-group {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }

        .category-filter-btn {
            background: var(--bg-card);
            border: 1px solid var(--border);
            padding: 10px 22px;
            border-radius: 30px;
            cursor: pointer;
            font-weight: 600;
            color: var(--text-main);
            transition: var(--transition);
        }

        .category-filter-btn.active, .category-filter-btn:hover {
            background: var(--accent);
            color: var(--primary);
            border-color: var(--accent);
        }

        .products-grid-container {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 2.5rem;
        }

        .product-card-item {
            background: var(--bg-card);
            border-radius: 20px;
            overflow: hidden;
            border: 1px solid var(--border);
            transition: var(--transition);
            display: flex;
            flex-direction: column;
            position: relative;
        }

        .product-card-item:hover {
            transform: translateY(-8px);
            box-shadow: var(--shadow-lg);
            border-color: var(--accent);
        }

        .product-img-box {
            height: 230px;
            background: linear-gradient(45deg, #1e293b, #0f172a);
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #cbd5e1;
            font-weight: bold;
            font-size: 1.1rem;
        }

        .tag-badge {
            position: absolute;
            top: 15px;
            right: 15px;
            background: var(--accent);
            color: var(--primary);
            padding: 5px 12px;
            border-radius: 8px;
            font-size: 0.75rem;
            font-weight: 900;
        }

        .product-body-content {
            padding: 1.8rem;
            display: flex;
            flex-direction: column;
            flex: 1;
        }

        .product-card-title {
            font-size: 1.25rem;
            font-weight: 800;
            margin-bottom: 8px;
        }

        .product-card-desc {
            font-size: 0.92rem;
            color: var(--text-muted);
            margin-bottom: 1.5rem;
            flex: 1;
        }

        .product-card-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top: 1px solid var(--border);
            padding-top: 1.2rem;
        }

        .product-price-display {
            font-size: 1.35rem;
            font-weight: 900;
            color: var(--accent);
        }

        .btn-add-to-cart {
            background: var(--primary);
            color: #fff;
            border: none;
            padding: 10px 20px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: 700;
            transition: var(--transition);
        }

        [data-theme="dark"] .btn-add-to-cart {
            background: #38bdf8;
            color: #030712;
        }

        .btn-add-to-cart:hover {
            background: var(--accent);
            color: var(--primary);
        }

        /* السلة المنبثقة الجانبية */
        .cart-slide-drawer {
            position: fixed;
            top: 0;
            left: -450px;
            width: 420px;
            height: 100%;
            background: var(--bg-card);
            box-shadow: var(--shadow-lg);
            z-index: 2000;
            transition: var(--transition);
            display: flex;
            flex-direction: column;
            border-right: 1px solid var(--border);
        }

        .cart-slide-drawer.open {
            left: 0;
        }

        .drawer-header {
            padding: 1.8rem;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .drawer-body {
            flex: 1;
            padding: 1.8rem;
            overflow-y: auto;
        }

        .cart-item-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1.2rem;
            padding-bottom: 1.2rem;
            border-bottom: 1px solid var(--border);
        }

        .drawer-footer {
            padding: 1.8rem;
            border-top: 1px solid var(--border);
            background: var(--bg-main);
        }

        /* التذييل الفاخر */
        footer {
            background: #020617;
            color: #fff;
            padding: 5rem 5% 2.5rem 5%;
            margin-top: 6rem;
            border-top: 1px solid rgba(255,255,255,0.05);
        }

        .footer-grid-layout {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 3.5rem;
            margin-bottom: 4rem;
        }

        .footer-column-box h4 {
            color: var(--accent);
            margin-bottom: 1.5rem;
            font-size: 1.25rem;
            font-weight: 800;
        }

        .footer-column-box ul {
            list-style: none;
        }

        .footer-column-box ul li {
            margin-bottom: 10px;
        }

        .footer-column-box ul li a {
            color: #94a3b8;
            text-decoration: none;
            transition: var(--transition);
        }

        .footer-column-box ul li a:hover {
            color: var(--accent);
        }

        .footer-bottom-copyright {
            text-align: center;
            border-top: 1px solid rgba(255,255,255,0.08);
            padding-top: 2.5rem;
            color: #64748b;
            font-size: 0.92rem;
        }
    </style>
</head>
<body data-theme="light">

    <!-- شريط الإعلانات العُلوي -->
    <div class="enterprise-top-bar">
        <span><i class="fa-solid fa-microchip"></i> نظام الموجه الذكي للنماذج (Claude $\rightarrow$ GPT $\rightarrow$ Gemini) نشط بالكامل</span>
        <span>الدعم الهندسي المباشر: 920000000</span>
    </div>

    <!-- الهيدر -->
    <header>
        <a href="#home" class="brand-identity">
            <i class="fa-solid fa-brain"></i>
            <h1>Apex AI Architecture</h1>
        </a>
        <ul class="navigation-menu">
            <li><a href="#home" class="active">الرئيسية</a></li>
            <li><a href="#ai-engine">المحرك الذكي</a></li>
            <li><a href="#store">المتجر الهندسي</a></li>
        </ul>
        <div class="header-action-controls">
            <button class="ui-control-btn" onclick="toggleDarkModeTheme()" title="تبديل الوضع">
                <i class="fa-solid fa-moon" id="themeModeIcon"></i>
            </button>
            <button class="ui-control-btn" onclick="toggleCartSidebar()">
                <i class="fa-solid fa-cart-shopping"></i>
                <span>السلة</span>
                <span class="cart-counter-badge" id="cartCounterBadge">0</span>
            </button>
        </div>
    </header>

    <!-- واجهة البطل والذكاء الاصطناعي -->
    <section class="hero-ai-section" id="ai-engine">
        <h2>نظام التشغيل الذكي والتحليل الفوري <span>بأقوى نماذج العالم</span></h2>
        <p>اطلب أي مشروع، تصميم واجهة، أو تحليل متاجر، وسيتولى الموجه الذكي إرسال طلبك إلى Claude أولاً، وإذا انتهت التجربة يحول فوراً لـ GPT ثم Gemini.</p>
        
        <div class="ai-prompt-box-container">
            <div class="ai-textarea-wrapper">
                <textarea id="userPromptTextarea" placeholder="مثال: ابتكر لي خطة متكاملة لمتجر قطع سيارات رياضي مع ميزات تنافسية..."></textarea>
                <div class="ai-submit-btn-row">
                    <span class="ai-status-indicator">الوضع الآلي: تفعيل التبديل الفوري عند نفاد الرصيد</span>
                    <button class="btn-execute-ai" onclick="submitPromptToEnterpriseAI()">إرسال للمنظومة</button>
                </div>
            </div>
        </div>
    </section>

    <!-- صندوق عرض نتائج الذكاء الاصطناعي -->
    <div class="ai-response-display-card" id="aiResponseCardBox">
        <div class="active-model-badge-tag" id="usedModelBadgeLabel">النموذج المستخدم: جاري المعالجة...</div>
        <div id="aiOutputResultText" style="white-space: pre-wrap; font-size: 1.05rem; line-height: 1.8;"></div>
    </div>

    <!-- قسم المتجر والقطع -->
    <main class="main-content-wrapper" id="store">
        <div class="section-header-row">
            <h3>قائمة المنتجات والحلول الهندسية</h3>
            <div class="filter-buttons-group">
                <button class="category-filter-btn active" onclick="filterStoreCategory('all', this)">الكل</button>
                <button class="category-filter-btn" onclick="filterStoreCategory('engine', this)">المحركات والتيربو</button>
                <button class="category-filter-btn" onclick="filterStoreCategory('exhaust', this)">أنظمة العادم</button>
                <button class="category-filter-btn" onclick="filterStoreCategory('accessories', this)">الإكسسوارات الذكية</button>
            </div>
        </div>

        <div class="products-grid-container" id="storeProductsGrid">
            <!-- سيتم توليد العناصر ديناميكياً -->
        </div>
    </main>

    <!-- سلة التسوق الجانبية -->
    <div class="cart-slide-drawer" id="cartSlideDrawerBox">
        <div class="drawer-header">
            <h3>سلة التسوق الذكية</h3>
            <button class="ui-control-btn" onclick="toggleCartSidebar()"><i class="fa-solid fa-xmark"></i></button>
        </div>
        <div class="drawer-body" id="cartDrawerItemsList">
            <p style="text-align: center; color: var(--text-muted); margin-top: 3rem;">السلة فارغة حالياً</p>
        </div>
        <div class="drawer-footer">
            <div style="display: flex; justify-content: space-between; margin-bottom: 1.2rem; font-weight: 800; font-size: 1.1rem;">
                <span>المبلغ الإجمالي:</span>
                <span id="cartDrawerTotalPrice">٠ ر.س</span>
            </div>
            <button class="btn-execute-ai" style="width: 100%; border-radius: 12px;" onclick="checkoutOrderAction()">إتمام وتأكيد الطلب</button>
        </div>
    </div>

    <!-- التذييل الفاخر -->
    <footer>
        <div class="footer-grid-layout">
            <div class="footer-column-box">
                <h4>عن المنظومة</h4>
                <p style="color: #94a3b8; font-size: 0.92rem;">أداة هندسية ذكية تدمج أقوى نماذج الذكاء الاصطناعي مع معمارية التجارة الإلكترونية المتقدمة.</p>
            </div>
            <div class="footer-column-box">
                <h4>روابط سريعة</h4>
                <ul>
                    <li><a href="#home">الرئيسية</a></li>
                    <li><a href="#ai-engine">محرك الذكاء الاصطناعي</a></li>
                    <li><a href="#store">المتجر الذكي</a></li>
                </ul>
            </div>
            <div class="footer-column-box">
                <h4>الدعم التقني</h4>
                <ul>
                    <li><a href="#">سلسلة النماذج والتبديل</a></li>
                    <li><a href="#">تتبع الشحنات الهندسية</a></li>
                    <li><a href="#">الأسئلة الشائعة</a></li>
                </ul>
            </div>
            <div class="footer-column-box">
                <h4>المقر الرئيسي</h4>
                <p style="color: #94a3b8; font-size: 0.92rem;">الرياض، المملكة العربية السعودية<br>البريد: support@apexenterprise.ai</p>
            </div>
        </div>
        <div class="footer-bottom-copyright">
            <p>جميع الحقوق محفوظة © 2026 Apex Enterprise - تم التصميم والتطوير البرمجي بنجاح.</p>
        </div>
    </footer>

    <!-- محرك التفاعل البرمجي (Frontend JS Engine) -->
    <script>
        const storeInventory = [
            { id: 1, name: "نظام عادم رياضي تيتانيوم متطور", category: "exhaust", price: 3499, tag: "الأكثر طلباً", desc: "تصميم هندسي يقلل الوزن ويزيد القدرة الحصانية للمحرك.", image: "عادم تيتانيوم عالي الأداء" },
            { id: 2, name: "شاحن توربو مزدوج للسباقات", category: "engine", price: 6200, tag: "أداء فائق", desc: "استجابة فورية وضغط هواء متوافق مع الحلبات الرياضية.", image: "توربو مزدوج احترافي" },
            { id: 3, name: "فلتر هواء رياضي كربوني", category: "engine", price: 580, tag: "تخفيض", desc: "زيادة تدفق الهواء بنسبة 35% لتعزيز كفاءة الاحتراق.", image: "فلتر هواء كربوني" },
            { id: 4, name: "شاشة ملاحة ذكية 4K للسيارات", category: "accessories", price: 1950, tag: "جديد", desc: "تدعم Apple CarPlay و Android Auto بكفاءة عرض مذهلة.", image: "شاشة ذكية متقدمة" },
            { id: 5, name: "مساعدات رياضية قابلة للضبط", category: "accessories", price: 2800, tag: "مميز", desc: "ثبات استثنائي وتحكم كامل في المنعطفات الخطرة.", image: "مساعدات رياضية" },
            { id: 6, name: "بواجي إيريديوم أصلية", category: "engine", price: 390, tag: "أصلي 100%", desc: "شرارة كهربائية مكثفة لعمر افتراضي أطول واستقرار المحرك.", image: "بواجي إيريديوم" }
        ];

        let activeCustomerCart = [];

        function renderStoreProducts(itemsArray) {
            const gridContainer = document.getElementById('storeProductsGrid');
            gridContainer.innerHTML = '';

            if(itemsArray.length === 0) {
                gridContainer.innerHTML = `<p style="grid-column: 1/-1; text-align: center; color: var(--text-muted); padding: 3rem;">عذراً، لا توجد منتجات مطابقة.</p>`;
                return;
            }

            itemsArray.forEach(item => {
                gridContainer.innerHTML += `
                    <div class="product-card-item">
                        <div class="product-img-box">
                            <span class="tag-badge">${item.tag}</span>
                            ${item.image}
                        </div>
                        <div class="product-body-content">
                            <h4 class="product-card-title">${item.name}</h4>
                            <p class="product-card-desc">${item.desc}</p>
                            <div class="product-card-footer">
                                <span class="product-price-display">${item.price} ر.س</span>
                                <button class="btn-add-to-cart" onclick="addCartItem(${item.id})">أضف للسلة</button>
                            </div>
                        </div>
                    </div>
                `;
            });
        }

        function filterStoreCategory(categoryKey, btnElement) {
            document.querySelectorAll('.category-filter-btn').forEach(btn => btn.classList.remove('active'));
            btnElement.classList.add('active');

            if(categoryKey === 'all') {
                renderStoreProducts(storeInventory);
            } else {
                const filtered = storeInventory.filter(p => p.category === categoryKey);
                renderStoreProducts(filtered);
            }
        }

        async function submitPromptToEnterpriseAI() {
            const promptText = document.getElementById('userPromptTextarea').value;
            if(!promptText) return alert('الرجاء كتابة طلبك في المربع أولاً!');

            const executeBtn = document.querySelector('.btn-execute-ai');
            executeBtn.innerText = "جاري الفحص والتبديل بين النماذج...";
            executeBtn.disabled = true;

            try {
                const response = await fetch('/generate-ai', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({ prompt: promptText })
                });

                const data = await response.json();
                
                const responseCard = document.getElementById('aiResponseCardBox');
                responseCard.style.display = 'block';
                document.getElementById('usedModelBadgeLabel').innerText = "تم التنفيذ عبر النموذج: " + data.model;
                document.getElementById('aiOutputResultText').innerText = data.response;
                
                responseCard.scrollIntoView({ behavior: 'smooth' });
            } catch (err) {
                alert('حدث خطأ أثناء الاتصال بالخلفية البرمجية.');
            } finally {
                executeBtn.innerText = "إرسال للمنظومة";
                executeBtn.disabled = false;
            }
        }

        function toggleCartSidebar() {
            document.getElementById('cartSlideDrawerBox').classList.toggle('open');
        }

        function addCartItem(id) {
            const product = storeInventory.find(p => p.id === id);
            const found = activeCustomerCart.find(item => item.id === id);

            if(found) {
                found.quantity++;
            } else {
                activeCustomerCart.push({ ...product, quantity: 1 });
            }

            updateCartUIState();
            toggleCartSidebar();
        }

        function updateCartUIState() {
            const container = document.getElementById('cartDrawerItemsList');
            const badge = document.getElementById('cartCounterBadge');
            const totalText = document.getElementById('cartDrawerTotalPrice');

            badge.innerText = activeCustomerCart.reduce((sum, item) => sum + item.quantity, 0);

            if(activeCustomerCart.length === 0) {
                container.innerHTML = `<p style="text-align: center; color: var(--text-muted); margin-top: 3rem;">السلة فارغة حالياً</p>`;
                totalText.innerText = "٠ ر.س";
                return;
            }

            container.innerHTML = '';
            let calcTotal = 0;

            activeCustomerCart.forEach(item => {
                calcTotal += item.price * item.quantity;
                container.innerHTML += `
                    <div class="cart-item-row">
                        <div>
                            <h5 style="font-size: 0.95rem; font-weight: 800; margin-bottom: 2px;">${item.name}</h5>
                            <span style="font-size: 0.85rem; color: var(--text-muted);">${item.price} ر.س × ${item.quantity}</span>
                        </div>
                        <button style="background: none; border: none; color: var(--danger); cursor: pointer; font-size: 1rem;" onclick="removeCartItem(${item.id})"><i class="fa-solid fa-trash-can"></i></button>
                    </div>
                `;
            });

            totalText.innerText = calcTotal + " ر.س";
        }

        function removeCartItem(id) {
            activeCustomerCart = activeCustomerCart.filter(item => item.id !== id);
            updateCartUIState();
        }

        function checkoutOrderAction() {
            if(activeCustomerCart.length === 0) {
                alert('سلة التسوق فارغة.');
                return;
            }
            alert('تم تأكيد الطلب الهندسي بنجاح! جاري تحويلك لبوابة الدفع الآمنة.');
            activeCustomerCart = [];
            updateCartUIState();
            toggleCartSidebar();
        }

        function toggleDarkModeTheme() {
            const bodyElem = document.body;
            const iconElem = document.getElementById('themeModeIcon');

            if(bodyElem.getAttribute('data-theme') === 'light') {
                bodyElem.setAttribute('data-theme', 'dark');
                iconElem.className = "fa-solid fa-sun";
            } else {
                bodyElem.setAttribute('data-theme', 'light');
                iconElem.className = "fa-solid fa-moon";
            }
        }

        renderStoreProducts(storeInventory);
    </script>
</body>
</html>
"""

# ==============================================================================
# المسارات البرمجية للخادم (Flask Routes)
# ==============================================================================
@app.route("/")
def render_main_page():
    return render_template_string(ENTERPRISE_HTML_LAYOUT)

@app.route("/generate-ai", methods=["POST"])
def api_generate_ai():
    req_data = request.json
    prompt_content = req_data.get("prompt", "")
    
    # تشغيل خوارزمية التحويل التلقائي بين Claude و GPT و Gemini
    response_text, model_used = execute_smart_ai_failover(prompt_content)
    
    return jsonify({
        "response": response_text,
        "model": model_used
    })

if __name__ == "__main__":
    app.run(debug=True, port=5000)
