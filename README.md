<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Virtual Machine Simulator</title>
    <style>
        /* ============================================
           CSS VARIABLES & ROOT STYLES
           ============================================ */
        :root {
            --primary-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            --secondary-gradient: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
            --accent-gradient: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            --dark-bg: #0a0a1a;
            --card-bg: rgba(255, 255, 255, 0.05);
            --glass-bg: rgba(255, 255, 255, 0.1);
            --glass-border: rgba(255, 255, 255, 0.2);
            --text-primary: #ffffff;
            --text-secondary: rgba(255, 255, 255, 0.7);
            --success: #00d4aa;
            --warning: #ffc107;
            --danger: #ff4757;
            --info: #4facfe;
        }

        /* ============================================
           RESET & BASE STYLES
           ============================================ */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: var(--dark-bg);
            color: var(--text-primary);
            min-height: 100vh;
            overflow-x: hidden;
        }

        /* Animated background */
        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: 
                radial-gradient(circle at 20% 80%, rgba(102, 126, 234, 0.15) 0%, transparent 50%),
                radial-gradient(circle at 80% 20%, rgba(118, 75, 162, 0.15) 0%, transparent 50%),
                radial-gradient(circle at 40% 40%, rgba(79, 172, 254, 0.1) 0%, transparent 40%);
            z-index: -1;
            animation: backgroundPulse 15s ease-in-out infinite;
        }

        @keyframes backgroundPulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }

        /* ============================================
           NAVIGATION STYLES
           ============================================ */
        .navbar {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: 70px;
            background: rgba(10, 10, 26, 0.8);
            backdrop-filter: blur(20px);
            border-bottom: 1px solid var(--glass-border);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 40px;
            z-index: 1000;
            transition: all 0.3s ease;
        }

        .navbar.scrolled {
            background: rgba(10, 10, 26, 0.95);
            box-shadow: 0 4px 30px rgba(0, 0, 0, 0.3);
        }

        .logo {
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 1.5rem;
            font-weight: 700;
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .logo-icon {
            width: 40px;
            height: 40px;
            background: var(--primary-gradient);
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 1.2rem;
        }

        .nav-links {
            display: flex;
            gap: 10px;
        }

        .nav-link {
            padding: 10px 24px;
            border: none;
            background: transparent;
            color: var(--text-secondary);
            font-size: 1rem;
            cursor: pointer;
            border-radius: 8px;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .nav-link::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: var(--primary-gradient);
            opacity: 0;
            transition: opacity 0.3s ease;
            z-index: -1;
        }

        .nav-link:hover {
            color: var(--text-primary);
        }

        .nav-link:hover::before {
            opacity: 0.2;
        }

        .nav-link.active {
            color: var(--text-primary);
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
        }

        /* ============================================
           PAGE CONTAINER STYLES
           ============================================ */
        .page-container {
            padding-top: 70px;
            min-height: 100vh;
        }

        .page {
            display: none;
            animation: fadeIn 0.5s ease;
        }

        .page.active {
            display: block;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        /* ============================================
           HOME PAGE STYLES
           ============================================ */
        .hero-section {
            min-height: calc(100vh - 70px);
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 60px 40px;
            position: relative;
        }

        .hero-content {
            max-width: 1200px;
            text-align: center;
            z-index: 1;
        }

        .hero-badge {
            display: inline-block;
            padding: 8px 20px;
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 50px;
            font-size: 0.9rem;
            color: var(--text-secondary);
            margin-bottom: 30px;
            backdrop-filter: blur(10px);
        }

        .hero-title {
            font-size: 4rem;
            font-weight: 800;
            line-height: 1.1;
            margin-bottom: 24px;
            background: linear-gradient(135deg, #fff 0%, rgba(255,255,255,0.7) 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .hero-title span {
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .hero-description {
            font-size: 1.25rem;
            color: var(--text-secondary);
            max-width: 700px;
            margin: 0 auto 40px;
            line-height: 1.8;
        }

        .hero-buttons {
            display: flex;
            gap: 20px;
            justify-content: center;
            flex-wrap: wrap;
        }

        .btn {
            padding: 14px 32px;
            border: none;
            border-radius: 12px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            display: inline-flex;
            align-items: center;
            gap: 10px;
        }

        .btn-primary {
            background: var(--primary-gradient);
            color: white;
            box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
        }

        .btn-primary:hover {
            transform: translateY(-3px);
            box-shadow: 0 8px 30px rgba(102, 126, 234, 0.6);
        }

        .btn-secondary {
            background: var(--glass-bg);
            color: var(--text-primary);
            border: 1px solid var(--glass-border);
            backdrop-filter: blur(10px);
        }

        .btn-secondary:hover {
            background: rgba(255, 255, 255, 0.15);
            transform: translateY(-3px);
        }

        /* Feature cards on home page */
        .features-section {
            padding: 80px 40px;
            max-width: 1400px;
            margin: 0 auto;
        }

        .section-title {
            text-align: center;
            font-size: 2.5rem;
            margin-bottom: 60px;
            background: linear-gradient(135deg, #fff 0%, rgba(255,255,255,0.7) 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .features-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 30px;
        }

        .feature-card {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 20px;
            padding: 40px 30px;
            backdrop-filter: blur(20px);
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .feature-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 4px;
            background: var(--primary-gradient);
            transform: scaleX(0);
            transition: transform 0.3s ease;
        }

        .feature-card:hover {
            transform: translateY(-10px);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
        }

        .feature-card:hover::before {
            transform: scaleX(1);
        }

        .feature-icon {
            width: 60px;
            height: 60px;
            background: var(--primary-gradient);
            border-radius: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            margin-bottom: 20px;
        }

        .feature-title {
            font-size: 1.3rem;
            margin-bottom: 12px;
            color: var(--text-primary);
        }

        .feature-desc {
            color: var(--text-secondary);
            line-height: 1.7;
        }

        /* ============================================
           SIMULATOR PAGE STYLES
           ============================================ */
        .simulator-container {
            max-width: 1600px;
            margin: 0 auto;
            padding: 40px;
        }

        .simulator-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 40px;
            flex-wrap: wrap;
            gap: 20px;
        }

        .simulator-title {
            font-size: 2rem;
            font-weight: 700;
        }

        .control-panel {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 16px;
            padding: 30px;
            backdrop-filter: blur(20px);
            margin-bottom: 40px;
        }

        .control-row {
            display: flex;
            gap: 30px;
            flex-wrap: wrap;
            align-items: flex-end;
        }

        .input-group {
            flex: 1;
            min-width: 200px;
        }

        .input-group label {
            display: block;
            margin-bottom: 8px;
            color: var(--text-secondary);
            font-size: 0.9rem;
        }

        .input-group input {
            width: 100%;
            padding: 14px 18px;
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid var(--glass-border);
            border-radius: 10px;
            color: var(--text-primary);
            font-size: 1rem;
            transition: all 0.3s ease;
        }

        .input-group input:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 20px rgba(102, 126, 234, 0.3);
        }

        .control-buttons {
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
        }

        /* Stats dashboard */
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 20px;
            margin-bottom: 40px;
        }

        .stat-card {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 16px;
            padding: 24px;
            backdrop-filter: blur(20px);
            position: relative;
            overflow: hidden;
        }

        .stat-card::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 3px;
            background: var(--primary-gradient);
        }

        .stat-card.success::after {
            background: linear-gradient(90deg, var(--success), #00ff88);
        }

        .stat-card.warning::after {
            background: linear-gradient(90deg, var(--warning), #ff9500);
        }

        .stat-card.info::after {
            background: var(--accent-gradient);
        }

        .stat-label {
            color: var(--text-secondary);
            font-size: 0.9rem;
            margin-bottom: 8px;
        }

        .stat-value {
            font-size: 2rem;
            font-weight: 700;
        }

        .stat-value.gradient {
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        /* CPU Usage visualization */
        .cpu-usage-container {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 16px;
            padding: 30px;
            margin-bottom: 40px;
            backdrop-filter: blur(20px);
        }

        .cpu-title {
            font-size: 1.3rem;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .cpu-meter {
            height: 30px;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 15px;
            overflow: hidden;
            position: relative;
        }

        .cpu-fill {
            height: 100%;
            background: var(--primary-gradient);
            border-radius: 15px;
            transition: width 0.5s ease;
            position: relative;
            overflow: hidden;
        }

        .cpu-fill::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: linear-gradient(
                90deg,
                transparent,
                rgba(255, 255, 255, 0.2),
                transparent
            );
            animation: shimmer 2s infinite;
        }

        @keyframes shimmer {
            0% { transform: translateX(-100%); }
            100% { transform: translateX(100%); }
        }

        .cpu-percentage {
            position: absolute;
            right: 15px;
            top: 50%;
            transform: translateY(-50%);
            font-weight: 600;
            color: var(--text-primary);
        }

        /* VM Grid */
        .vm-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 25px;
        }

        .vm-card {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 20px;
            padding: 25px;
            backdrop-filter: blur(20px);
            transition: all 0.3s ease;
            animation: slideIn 0.5s ease;
        }

        @keyframes slideIn {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
        }

        .vm-card:hover {
            border-color: rgba(102, 126, 234, 0.5);
            box-shadow: 0 10px 40px rgba(102, 126, 234, 0.2);
        }

        .vm-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 1px solid var(--glass-border);
        }

        .vm-name {
            font-size: 1.2rem;
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .vm-icon {
            width: 40px;
            height: 40px;
            background: var(--accent-gradient);
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .vm-status {
            padding: 6px 14px;
            border-radius: 20px;
            font-size: 0.85rem;
            font-weight: 600;
        }

        .vm-status.running {
            background: rgba(0, 212, 170, 0.2);
            color: var(--success);
        }

        .vm-status.idle {
            background: rgba(255, 193, 7, 0.2);
            color: var(--warning);
        }

        .vm-status.completed {
            background: rgba(79, 172, 254, 0.2);
            color: var(--info);
        }

        .process-list {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .process-item {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 12px;
            padding: 15px;
            transition: all 0.3s ease;
        }

        .process-item.active {
            background: rgba(102, 126, 234, 0.2);
            border: 1px solid rgba(102, 126, 234, 0.4);
        }

        .process-item.completed {
            opacity: 0.6;
        }

        .process-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }

        .process-name {
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .process-status {
            font-size: 0.8rem;
            padding: 4px 10px;
            border-radius: 12px;
        }

        .process-status.running {
            background: rgba(0, 212, 170, 0.2);
            color: var(--success);
        }

        .process-status.waiting {
            background: rgba(255, 193, 7, 0.2);
            color: var(--warning);
        }

        .process-status.completed {
            background: rgba(79, 172, 254, 0.2);
            color: var(--info);
        }

        .process-progress {
            height: 8px;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 4px;
            overflow: hidden;
        }

        .process-progress-fill {
            height: 100%;
            background: var(--primary-gradient);
            border-radius: 4px;
            transition: width 0.3s ease;
        }

        .process-info {
            display: flex;
            justify-content: space-between;
            margin-top: 8px;
            font-size: 0.85rem;
            color: var(--text-secondary);
        }

        /* Chart container */
        .chart-container {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 16px;
            padding: 30px;
            margin-top: 40px;
            backdrop-filter: blur(20px);
        }

        .chart-title {
            font-size: 1.3rem;
            margin-bottom: 20px;
        }

        #cpuChart {
            max-height: 300px;
        }

        /* ============================================
           FLOWCHART PAGE STYLES
           ============================================ */
        .flowchart-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 40px;
        }

        .flowchart-title {
            text-align: center;
            font-size: 2.5rem;
            margin-bottom: 20px;
        }

        .flowchart-subtitle {
            text-align: center;
            color: var(--text-secondary);
            margin-bottom: 60px;
            font-size: 1.1rem;
        }

        .flowchart {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
        }

        .flow-step {
            width: 100%;
            max-width: 500px;
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 20px;
            padding: 30px;
            backdrop-filter: blur(20px);
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
        }

        .flow-step:hover {
            transform: scale(1.02);
            border-color: rgba(102, 126, 234, 0.5);
            box-shadow: 0 10px 40px rgba(102, 126, 234, 0.3);
        }

        .flow-step::before {
            content: '';
            position: absolute;
            left: 50%;
            bottom: -20px;
            transform: translateX(-50%);
            width: 2px;
            height: 20px;
            background: var(--primary-gradient);
        }

        .flow-step:last-child::before {
            display: none;
        }

        .flow-step-number {
            position: absolute;
            top: -15px;
            left: 30px;
            width: 30px;
            height: 30px;
            background: var(--primary-gradient);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            font-size: 0.9rem;
        }

        .flow-step-icon {
            width: 60px;
            height: 60px;
            background: rgba(102, 126, 234, 0.2);
            border-radius: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            margin-bottom: 15px;
        }

        .flow-step-title {
            font-size: 1.3rem;
            font-weight: 600;
            margin-bottom: 10px;
        }

        .flow-step-desc {
            color: var(--text-secondary);
            font-size: 0.95rem;
        }

        .flow-arrow {
            font-size: 2rem;
            color: var(--text-secondary);
            animation: bounce 1s infinite;
        }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(5px); }
        }

        /* Modal styles */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            backdrop-filter: blur(10px);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 2000;
            opacity: 0;
            visibility: hidden;
            transition: all 0.3s ease;
        }

        .modal-overlay.active {
            opacity: 1;
            visibility: visible;
        }

        .modal {
            background: var(--dark-bg);
            border: 1px solid var(--glass-border);
            border-radius: 24px;
            padding: 40px;
            max-width: 600px;
            width: 90%;
            max-height: 80vh;
            overflow-y: auto;
            transform: scale(0.9);
            transition: transform 0.3s ease;
        }

        .modal-overlay.active .modal {
            transform: scale(1);
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 25px;
        }

        .modal-title {
            font-size: 1.5rem;
            font-weight: 700;
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .modal-close {
            width: 40px;
            height: 40px;
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 10px;
            color: var(--text-primary);
            font-size: 1.2rem;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .modal-close:hover {
            background: rgba(255, 71, 87, 0.2);
            border-color: var(--danger);
        }

        .modal-content {
            color: var(--text-secondary);
            line-height: 1.8;
        }

        .modal-content h4 {
            color: var(--text-primary);
            margin: 20px 0 10px;
        }

        .modal-content ul {
            margin-left: 20px;
        }

        .modal-content li {
            margin: 8px 0;
        }

        /* ============================================
           RESPONSIVE STYLES
           ============================================ */
        @media (max-width: 768px) {
            .navbar {
                padding: 0 20px;
            }

            .logo span {
                display: none;
            }

            .nav-link {
                padding: 8px 16px;
                font-size: 0.9rem;
            }

            .hero-title {
                font-size: 2.5rem;
            }

            .hero-description {
                font-size: 1rem;
            }

            .simulator-container,
            .flowchart-container,
            .features-section {
                padding: 20px;
            }

            .vm-grid {
                grid-template-columns: 1fr;
            }

            .control-row {
                flex-direction: column;
            }

            .input-group {
                width: 100%;
            }

            .stats-grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        /* ============================================
           UTILITY CLASSES
           ============================================ */
        .hidden {
            display: none !important;
        }

        .text-gradient {
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        /* Toast notifications */
        .toast-container {
            position: fixed;
            bottom: 30px;
            right: 30px;
            z-index: 3000;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .toast {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 12px;
            padding: 16px 24px;
            backdrop-filter: blur(20px);
            display: flex;
            align-items: center;
            gap: 12px;
            animation: slideInRight 0.3s ease;
            min-width: 300px;
        }

        @keyframes slideInRight {
            from { opacity: 0; transform: translateX(100px); }
            to { opacity: 1; transform: translateX(0); }
        }

        .toast.success {
            border-color: var(--success);
        }

        .toast.info {
            border-color: var(--info);
        }

        .toast.warning {
            border-color: var(--warning);
        }

        .toast-icon {
            font-size: 1.2rem;
        }

        .toast-message {
            flex: 1;
        }

        /* Pulse animation for active elements */
        .pulse {
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0%, 100% { box-shadow: 0 0 0 0 rgba(102, 126, 234, 0.4); }
            50% { box-shadow: 0 0 0 10px rgba(102, 126, 234, 0); }
        }
    </style>
</head>
<body>
    <!-- ============================================
         NAVIGATION BAR
         ============================================ -->
    <nav class="navbar" id="navbar">
        <div class="logo">
            <div class="logo-icon">⚡</div>
            <span>VM Simulator</span>
        </div>
        <div class="nav-links">
            <button class="nav-link active" data-page="home">🏠 Home</button>
            <button class="nav-link" data-page="simulator">🖥️ Simulator</button>
            <button class="nav-link" data-page="flowchart">📊 Flowchart</button>
        </div>
    </nav>

    <!-- ============================================
         PAGE CONTAINER
         ============================================ -->
    <div class="page-container">
        <!-- HOME PAGE -->
        <div class="page active" id="home">
            <section class="hero-section">
                <div class="hero-content">
                   
                    <h1 class="hero-title">
                        Simple <span>Virtual Machine</span><br>Simulator
                    </h1>
                    <p class="hero-description">
                        Experience the power of virtualization technology with our interactive simulator. 
                        Create virtual machines, manage processes, and visualize Round Robin scheduling 
                        in real-time with stunning visual feedback.
                    </p>
                    <div class="hero-buttons">
                        <button class="btn btn-primary" onclick="navigateTo('simulator')">
                            🎮 Launch Simulator
                        </button>
                        <button class="btn btn-secondary" onclick="navigateTo('flowchart')">
                            📖 View Flowchart
                        </button>
                    </div>
                </div>
            </section>

        </div>

        <!-- SIMULATOR PAGE -->
        <div class="page" id="simulator">
            <div class="simulator-container">
                <div class="simulator-header">
                    <h1 class="simulator-title">🖥️ VM Simulator Dashboard</h1>
                </div>

                <!-- Control Panel -->
                <div class="control-panel">
                    <div class="control-row">
                        <div class="input-group">
                            <label for="vmCount">Number of Virtual Machines</label>
                            <input type="number" id="vmCount" min="1" max="6" value="3" placeholder="1-6 VMs">
                        </div>
                        <div class="input-group">
                            <label for="processCount">Processes per VM</label>
                            <input type="number" id="processCount" min="1" max="5" value="3" placeholder="1-5 processes">
                        </div>
                        <div class="input-group">
                            <label for="timeQuantum">Time Quantum (ms)</label>
                            <input type="number" id="timeQuantum" min="100" max="2000" value="500" placeholder="100-2000ms">
                        </div>
                        <div class="control-buttons">
                            <button class="btn btn-primary" id="startBtn" onclick="startSimulation()">
                                ▶️ Start Simulation
                            </button>
                            <button class="btn btn-secondary" id="pauseBtn" onclick="pauseSimulation()" disabled>
                                ⏸️ Pause
                            </button>
                            <button class="btn btn-secondary" id="resetBtn" onclick="resetSimulation()">
                                🔄 Reset
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Stats Dashboard -->
                <div class="stats-grid">
                    <div class="stat-card">
                        <div class="stat-label">Total VMs</div>
                        <div class="stat-value gradient" id="totalVms">0</div>
                    </div>
                    <div class="stat-card success">
                        <div class="stat-label">Active Processes</div>
                        <div class="stat-value" id="activeProcesses">0</div>
                    </div>
                    <div class="stat-card warning">
                        <div class="stat-label">Completed Processes</div>
                        <div class="stat-value" id="completedProcesses">0</div>
                    </div>
                    <div class="stat-card info">
                        <div class="stat-label">Simulation Time</div>
                        <div class="stat-value" id="simTime">0s</div>
                    </div>
                </div>

                <!-- CPU Usage Visualization -->
                <div class="cpu-usage-container">
                    <div class="cpu-title">
                        <span>📊</span> Overall CPU Utilization
                    </div>
                    <div class="cpu-meter">
                        <div class="cpu-fill" id="cpuFill" style="width: 0%"></div>
                        <span class="cpu-percentage" id="cpuPercentage">0%</span>
                    </div>
                </div>

                <!-- VM Grid -->
                <div class="vm-grid" id="vmGrid">
                    <!-- VMs will be dynamically generated here -->
                </div>

                <!-- CPU Usage Chart -->
                <div class="chart-container">
                    <h3 class="chart-title">📈 CPU Usage Over Time</h3>
                    <canvas id="cpuChart"></canvas>
                </div>
            </div>
        </div>

        <!-- FLOWCHART PAGE -->
        <div class="page" id="flowchart">
            <div class="flowchart-container">
                <h1 class="flowchart-title">System Architecture Flowchart</h1>
                <p class="flowchart-subtitle">Click on each step to learn more about the process</p>
                
                <div class="flowchart">
                    <div class="flow-step" onclick="showModal('vmCreation')">
                        <div class="flow-step-number">1</div>
                        <div class="flow-step-icon">🖥️</div>
                        <h3 class="flow-step-title">VM Creation</h3>
                        <p class="flow-step-desc">Initialize virtual machines with allocated resources</p>
                    </div>
                    
                    <div class="flow-arrow">↓</div>
                    
                    <div class="flow-step" onclick="showModal('processAllocation')">
                        <div class="flow-step-number">2</div>
                        <div class="flow-step-icon">📋</div>
                        <h3 class="flow-step-title">Process Allocation</h3>
                        <p class="flow-step-desc">Assign processes to virtual machines with burst times</p>
                    </div>
                    
                    <div class="flow-arrow">↓</div>
                    
                    <div class="flow-step" onclick="showModal('scheduling')">
                        <div class="flow-step-number">3</div>
                        <div class="flow-step-icon">🔄</div>
                        <h3 class="flow-step-title">Round Robin Scheduling</h3>
                        <p class="flow-step-desc">Apply time quantum-based CPU scheduling algorithm</p>
                    </div>
                    
                    <div class="flow-arrow">↓</div>
                    
                    <div class="flow-step" onclick="showModal('execution')">
                        <div class="flow-step-number">4</div>
                        <div class="flow-step-icon">⚡</div>
                        <h3 class="flow-step-title">Process Execution</h3>
                        <p class="flow-step-desc">Execute processes within their time quantum slice</p>
                    </div>
                    
                    <div class="flow-arrow">↓</div>
                    
                    <div class="flow-step" onclick="showModal('contextSwitch')">
                        <div class="flow-step-number">5</div>
                        <div class="flow-step-icon">🔀</div>
                        <h3 class="flow-step-title">Context Switching</h3>
                        <p class="flow-step-desc">Save state and switch to next process in ready queue</p>
                    </div>
                    
                    <div class="flow-arrow">↓</div>
                    
                    <div class="flow-step" onclick="showModal('completion')">
                        <div class="flow-step-number">6</div>
                        <div class="flow-step-icon">✅</div>
                        <h3 class="flow-step-title">Completion & Reporting</h3>
                        <p class="flow-step-desc">Track completed processes and generate statistics</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- ============================================
         MODAL OVERLAY
         ============================================ -->
    <div class="modal-overlay" id="modalOverlay" onclick="closeModal(event)">
        <div class="modal" onclick="event.stopPropagation()">
            <div class="modal-header">
                <h2 class="modal-title" id="modalTitle">Modal Title</h2>
                <button class="modal-close" onclick="closeModal()">&times;</button>
            </div>
            <div class="modal-content" id="modalContent">
                <!-- Content will be dynamically inserted -->
            </div>
        </div>
    </div>

    <!-- ============================================
         TOAST CONTAINER
         ============================================ -->
    <div class="toast-container" id="toastContainer"></div>

    <!-- ============================================
         JAVASCRIPT
         ============================================ -->
    <script>
        // ============================================
        // GLOBAL VARIABLES & STATE
        // ============================================
        let virtualMachines = [];
        let simulationInterval = null;
        let isPaused = false;
        let simulationTime = 0;
        let cpuUsageHistory = [];
        let chartInstance = null;
        let audioContext = null;

        // Modal content data
        const modalData = {
            vmCreation: {
                title: '🖥️ Virtual Machine Creation',
                content: `
                    <p>Virtual Machine creation is the first step in the virtualization process. During this phase:</p>
                    <h4>Key Operations:</h4>
                    <ul>
                        <li><strong>Resource Allocation:</strong> CPU cores, memory, and storage are allocated to each VM</li>
                        <li><strong>Hypervisor Setup:</strong> The virtualization layer initializes the VM container</li>
                        <li><strong>OS Initialization:</strong> Guest operating system environment is prepared</li>
                        <li><strong>Network Configuration:</strong> Virtual network interfaces are created</li>
                    </ul>
                    <h4>In This Simulator:</h4>
                    <p>We create lightweight VM objects that maintain their own process queues and state. Each VM operates independently while sharing the simulated CPU resources through the scheduler.</p>
                `
            },
            processAllocation: {
                title: '📋 Process Allocation',
                content: `
                    <p>Process allocation involves assigning computational tasks to virtual machines based on various factors:</p>
                    <h4>Allocation Factors:</h4>
                    <ul>
                        <li><strong>Burst Time:</strong> Estimated CPU time required for process completion</li>
                        <li><strong>Priority Level:</strong> Importance of the process (in advanced schedulers)</li>
                        <li><strong>Resource Requirements:</strong> Memory and I/O needs</li>
                        <li><strong>Load Balancing:</strong> Distributing work evenly across VMs</li>
                    </ul>
                    <h4>Process States:</h4>
                    <p>Each process can be in one of three states: <strong>Waiting</strong> (in ready queue), <strong>Running</strong> (currently executing), or <strong>Completed</strong> (finished execution).</p>
                `
            },
            scheduling: {
                title: '🔄 Round Robin Scheduling',
                content: `
                    <p>Round Robin is a preemptive scheduling algorithm designed for time-sharing systems:</p>
                    <h4>Algorithm Characteristics:</h4>
                    <ul>
                        <li><strong>Time Quantum:</strong> Fixed time slice allocated to each process</li>
                        <li><strong>Fairness:</strong> All processes get equal CPU time in rotation</li>
                        <li><strong>No Starvation:</strong> Every process eventually gets CPU time</li>
                        <li><strong>Preemptive:</strong> Processes are interrupted after their quantum expires</li>
                    </ul>
                    <h4>Trade-offs:</h4>
                    <p>Smaller quantum = More responsive but higher context switch overhead<br>
                    Larger quantum = Less overhead but longer waiting times</p>
                `
            },
            execution: {
                title: '⚡ Process Execution',
                content: `
                    <p>During execution, the CPU performs the actual computation for the running process:</p>
                    <h4>Execution Cycle:</h4>
                    <ul>
                        <li><strong>Fetch:</strong> Load process state and instructions</li>
                        <li><strong>Execute:</strong> Perform computational operations</li>
                        <li><strong>Update:</strong> Modify remaining burst time</li>
                        <li><strong>Check:</strong> Determine if quantum expired or process completed</li>
                    </ul>
                    <h4>In This Simulator:</h4>
                    <p>We simulate execution by decrementing the remaining burst time at each tick. The visual progress bar shows the completion percentage of each process.</p>
                `
            },
            contextSwitch: {
                title: '🔀 Context Switching',
                content: `
                    <p>Context switching occurs when the CPU switches from one process to another:</p>
                    <h4>What Gets Saved:</h4>
                    <ul>
                        <li><strong>Program Counter:</strong> Current instruction position</li>
                        <li><strong>CPU Registers:</strong> All register values</li>
                        <li><strong>Memory Management Info:</strong> Page tables, segment tables</li>
                        <li><strong>I/O Status:</strong> Open files, device states</li>
                    </ul>
                    <h4>Performance Impact:</h4>
                    <p>Context switches have overhead (typically 1-1000 microseconds). Frequent switches reduce throughput but improve responsiveness.</p>
                `
            },
            completion: {
                title: '✅ Completion & Reporting',
                content: `
                    <p>When processes complete, the system collects performance metrics:</p>
                    <h4>Metrics Tracked:</h4>
                    <ul>
                        <li><strong>Turnaround Time:</strong> Total time from submission to completion</li>
                        <li><strong>Waiting Time:</strong> Time spent in ready queue</li>
                        <li><strong>CPU Utilization:</strong> Percentage of time CPU was active</li>
                        <li><strong>Throughput:</strong> Number of processes completed per unit time</li>
                    </ul>
                    <h4>Resource Cleanup:</h4>
                    <p>Completed processes release their resources, which are then available for new processes or VM scaling operations.</p>
                `
            }
        };

        // ============================================
        // NAVIGATION FUNCTIONS
        // ============================================
        
        /**
         * Navigate to a specific page
         * @param {string} pageName - The page to navigate to
         */
        function navigateTo(pageName) {
            // Update nav links
            document.querySelectorAll('.nav-link').forEach(link => {
                link.classList.remove('active');
                if (link.dataset.page === pageName) {
                    link.classList.add('active');
                }
            });

            // Update pages
            document.querySelectorAll('.page').forEach(page => {
                page.classList.remove('active');
            });
            document.getElementById(pageName).classList.add('active');

            // Initialize chart when navigating to simulator
            if (pageName === 'simulator' && !chartInstance) {
                initChart();
            }
        }

        // Set up navigation event listeners
        document.querySelectorAll('.nav-link').forEach(link => {
            link.addEventListener('click', () => {
                navigateTo(link.dataset.page);
            });
        });

        // Navbar scroll effect
        window.addEventListener('scroll', () => {
            const navbar = document.getElementById('navbar');
            if (window.scrollY > 50) {
                navbar.classList.add('scrolled');
            } else {
                navbar.classList.remove('scrolled');
            }
        });

        // ============================================
        // TOAST NOTIFICATION SYSTEM
        // ============================================
        
        /**
         * Show a toast notification
         * @param {string} message - The message to display
         * @param {string} type - Type of toast (success, info, warning)
         */
        function showToast(message, type = 'info') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = `toast ${type}`;
            
            const icons = {
                success: '✅',
                info: 'ℹ️',
                warning: '⚠️'
            };
            
            toast.innerHTML = `
                <span class="toast-icon">${icons[type]}</span>
                <span class="toast-message">${message}</span>
            `;
            
            container.appendChild(toast);
            
            // Play sound effect
            playSound(type);
            
            // Remove toast after 3 seconds
            setTimeout(() => {
                toast.style.animation = 'slideInRight 0.3s ease reverse';
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        // ============================================
        // SOUND EFFECTS
        // ============================================
        
        /**
         * Initialize audio context and play sounds
         * @param {string} type - Type of sound to play
         */
        function playSound(type) {
            if (!audioContext) {
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
            }
            
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            const frequencies = {
                success: 800,
                info: 600,
                warning: 400,
                tick: 1200
            };
            
            oscillator.frequency.value = frequencies[type] || 600;
            oscillator.type = 'sine';
            
            gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.1);
        }

        // ============================================
        // MODAL FUNCTIONS
        // ============================================
        
        /**
         * Show modal with specified content
         * @param {string} contentKey - Key for modal content
         */
        function showModal(contentKey) {
            const data = modalData[contentKey];
            if (!data) return;
            
            document.getElementById('modalTitle').textContent = data.title;
            document.getElementById('modalContent').innerHTML = data.content;
            document.getElementById('modalOverlay').classList.add('active');
            playSound('info');
        }

        /**
         * Close the modal
         * @param {Event} event - Click event
         */
        function closeModal(event) {
            if (!event || event.target === document.getElementById('modalOverlay')) {
                document.getElementById('modalOverlay').classList.remove('active');
            }
        }

        // Close modal on Escape key
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') closeModal();
        });

        // ============================================
        // CHART INITIALIZATION
        // ============================================
        
        /**
         * Initialize the CPU usage chart
         */
        function initChart() {
            const ctx = document.getElementById('cpuChart').getContext('2d');
            
            // Create gradient for chart
            const gradient = ctx.createLinearGradient(0, 0, 0, 300);
            gradient.addColorStop(0, 'rgba(102, 126, 234, 0.5)');
            gradient.addColorStop(1, 'rgba(102, 126, 234, 0)');
            
            chartInstance = {
                ctx: ctx,
                gradient: gradient,
                data: [],
                labels: [],
                maxPoints: 50
            };
            
            drawChart();
        }

        /**
         * Draw/update the CPU usage chart
         */
        function drawChart() {
            if (!chartInstance) return;
            
            const ctx = chartInstance.ctx;
            const canvas = ctx.canvas;
            const width = canvas.width = canvas.offsetWidth;
            const height = canvas.height = canvas.offsetHeight;
            
            ctx.clearRect(0, 0, width, height);
            
            // Draw grid
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.1)';
            ctx.lineWidth = 1;
            
            for (let i = 0; i <= 4; i++) {
                const y = (height / 4) * i;
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(width, y);
                ctx.stroke();
            }
            
            // Draw data
            if (chartInstance.data.length > 1) {
                const data = chartInstance.data;
                const stepX = width / (chartInstance.maxPoints - 1);
                
                // Fill area
                ctx.beginPath();
                ctx.moveTo(0, height);
                
                for (let i = 0; i < data.length; i++) {
                    const x = i * stepX;
                    const y = height - (data[i] / 100 * height);
                    ctx.lineTo(x, y);
                }
                
                ctx.lineTo((data.length - 1) * stepX, height);
                ctx.closePath();
                ctx.fillStyle = chartInstance.gradient;
                ctx.fill();
                
                // Draw line
                ctx.beginPath();
                ctx.moveTo(0, height - (data[0] / 100 * height));
                
                for (let i = 1; i < data.length; i++) {
                    const x = i * stepX;
                    const y = height - (data[i] / 100 * height);
                    ctx.lineTo(x, y);
                }
                
                ctx.strokeStyle = '#667eea';
                ctx.lineWidth = 3;
                ctx.stroke();
                
                // Draw points
                for (let i = 0; i < data.length; i++) {
                    const x = i * stepX;
                    const y = height - (data[i] / 100 * height);
                    
                    ctx.beginPath();
                    ctx.arc(x, y, 4, 0, Math.PI * 2);
                    ctx.fillStyle = '#667eea';
                    ctx.fill();
                }
            }
            
            // Draw labels
            ctx.fillStyle = 'rgba(255, 255, 255, 0.5)';
            ctx.font = '12px sans-serif';
            ctx.textAlign = 'left';
            ctx.fillText('100%', 5, 15);
            ctx.fillText('50%', 5, height / 2 + 5);
            ctx.fillText('0%', 5, height - 5);
        }

        /**
         * Update chart with new data point
         * @param {number} value - CPU usage percentage
         */
        function updateChart(value) {
            if (!chartInstance) return;
            
            chartInstance.data.push(value);
            
            if (chartInstance.data.length > chartInstance.maxPoints) {
                chartInstance.data.shift();
            }
            
            drawChart();
        }

        // ============================================
        // VIRTUAL MACHINE CLASS
        // ============================================
        
        /**
         * Virtual Machine class representing a single VM
         */
        class VirtualMachine {
            constructor(id, processCount) {
                this.id = id;
                this.name = `VM-${String(id).padStart(2, '0')}`;
                this.processes = [];
                this.status = 'idle';
                this.currentProcessIndex = 0;
                
                // Create processes with random burst times
                for (let i = 0; i < processCount; i++) {
                    this.processes.push({
                        id: `P${id}-${i + 1}`,
                        name: `Process ${i + 1}`,
                        burstTime: Math.floor(Math.random() * 5000) + 2000, // 2-7 seconds
                        remainingTime: 0,
                        status: 'waiting',
                        progress: 0
                    });
                    // Set remaining time equal to burst time initially
                    this.processes[i].remainingTime = this.processes[i].burstTime;
                }
            }

            /**
             * Get the next process to run
             * @returns {Object|null} Next process or null if all completed
             */
            getNextProcess() {
                const waitingProcesses = this.processes.filter(p => p.status !== 'completed');
                if (waitingProcesses.length === 0) {
                    this.status = 'completed';
                    return null;
                }
                
                // Find next waiting process using round robin
                let checked = 0;
                while (checked < this.processes.length) {
                    const process = this.processes[this.currentProcessIndex];
                    this.currentProcessIndex = (this.currentProcessIndex + 1) % this.processes.length;
                    
                    if (process.status !== 'completed') {
                        return process;
                    }
                    checked++;
                }
                
                return null;
            }

            /**
             * Execute one tick of the current process
             * @param {number} quantum - Time quantum in ms
             * @returns {boolean} True if process executed
             */
            tick(quantum) {
                // Set all non-completed processes to waiting
                this.processes.forEach(p => {
                    if (p.status !== 'completed') {
                        p.status = 'waiting';
                    }
                });

                const process = this.getNextProcess();
                if (!process) return false;

                this.status = 'running';
                process.status = 'running';
                
                // Decrease remaining time by quantum
                const executeTime = Math.min(quantum, process.remainingTime);
                process.remainingTime -= executeTime;
                
                // Calculate progress
                process.progress = ((process.burstTime - process.remainingTime) / process.burstTime) * 100;
                
                // Check if process completed
                if (process.remainingTime <= 0) {
                    process.status = 'completed';
                    process.progress = 100;
                    playSound('success');
                }
                
                return true;
            }

            /**
             * Check if all processes are completed
             * @returns {boolean} True if all completed
             */
            isCompleted() {
                return this.processes.every(p => p.status === 'completed');
            }
        }

        // ============================================
        // SIMULATION FUNCTIONS
        // ============================================
        
        /**
         * Initialize and start the simulation
         */
        function startSimulation() {
            if (simulationInterval) {
                // Resume if paused
                if (isPaused) {
                    isPaused = false;
                    document.getElementById('pauseBtn').textContent = '⏸️ Pause';
                    showToast('Simulation resumed', 'info');
                    return;
                }
                return;
            }

            // Get input values
            const vmCount = parseInt(document.getElementById('vmCount').value) || 3;
            const processCount = parseInt(document.getElementById('processCount').value) || 3;
            const timeQuantum = parseInt(document.getElementById('timeQuantum').value) || 500;

            // Validate inputs
            if (vmCount < 1 || vmCount > 6) {
                showToast('Please enter 1-6 VMs', 'warning');
                return;
            }
            if (processCount < 1 || processCount > 5) {
                showToast('Please enter 1-5 processes per VM', 'warning');
                return;
            }
            if (timeQuantum < 100 || timeQuantum > 2000) {
                showToast('Time quantum must be 100-2000ms', 'warning');
                return;
            }

            // Create VMs
            virtualMachines = [];
            for (let i = 1; i <= vmCount; i++) {
                virtualMachines.push(new VirtualMachine(i, processCount));
            }

            // Reset state
            simulationTime = 0;
            cpuUsageHistory = [];
            isPaused = false;

            // Update UI
            document.getElementById('startBtn').disabled = true;
            document.getElementById('pauseBtn').disabled = false;
            
            // Render initial state
            renderVMs();
            updateStats();
            
            showToast(`Simulation started with ${vmCount} VMs`, 'success');

            // Start simulation loop
            let currentVMIndex = 0;
            
            simulationInterval = setInterval(() => {
                if (isPaused) return;

                // Check if all VMs completed
                if (virtualMachines.every(vm => vm.isCompleted())) {
                    stopSimulation();
                    showToast('All processes completed!', 'success');
                    return;
                }

                // Execute one quantum on current VM
                let executed = false;
                let attempts = 0;
                
                while (!executed && attempts < virtualMachines.length) {
                    const vm = virtualMachines[currentVMIndex];
                    executed = vm.tick(timeQuantum);
                    currentVMIndex = (currentVMIndex + 1) % virtualMachines.length;
                    attempts++;
                }

                // Update simulation time
                simulationTime += timeQuantum;

                // Calculate CPU usage
                const activeProcesses = virtualMachines.reduce((acc, vm) => 
                    acc + vm.processes.filter(p => p.status !== 'completed').length, 0);
                const totalProcesses = virtualMachines.reduce((acc, vm) => 
                    acc + vm.processes.length, 0);
                const cpuUsage = totalProcesses > 0 ? 
                    ((totalProcesses - virtualMachines.reduce((acc, vm) => 
                        acc + vm.processes.filter(p => p.status === 'completed').length, 0)) / totalProcesses) * 100 : 0;

                // Update chart
                updateChart(cpuUsage);

                // Update UI
                renderVMs();
                updateStats();
                
                // Play tick sound occasionally
                if (Math.random() < 0.1) {
                    playSound('tick');
                }
                
            }, timeQuantum / 2); // Run faster than real-time for better visualization
        }

        /**
         * Pause/resume the simulation
         */
        function pauseSimulation() {
            isPaused = !isPaused;
            document.getElementById('pauseBtn').textContent = isPaused ? '▶️ Resume' : '⏸️ Pause';
            showToast(isPaused ? 'Simulation paused' : 'Simulation resumed', 'info');
        }

        /**
         * Stop the simulation completely
         */
        function stopSimulation() {
            if (simulationInterval) {
                clearInterval(simulationInterval);
                simulationInterval = null;
            }
            document.getElementById('startBtn').disabled = false;
            document.getElementById('pauseBtn').disabled = true;
        }

        /**
         * Reset the simulation
         */
        function resetSimulation() {
            stopSimulation();
            virtualMachines = [];
            simulationTime = 0;
            cpuUsageHistory = [];
            isPaused = false;
            
            // Reset chart
            if (chartInstance) {
                chartInstance.data = [];
                drawChart();
            }
            
            // Clear VM grid
            document.getElementById('vmGrid').innerHTML = '';
            
            // Reset stats
            document.getElementById('totalVms').textContent = '0';
            document.getElementById('activeProcesses').textContent = '0';
            document.getElementById('completedProcesses').textContent = '0';
            document.getElementById('simTime').textContent = '0s';
            document.getElementById('cpuFill').style.width = '0%';
            document.getElementById('cpuPercentage').textContent = '0%';
            
            showToast('Simulation reset', 'info');
        }

        /**
         * Render all VMs to the grid
         */
        function renderVMs() {
            const grid = document.getElementById('vmGrid');
            grid.innerHTML = '';
            
            virtualMachines.forEach(vm => {
                const vmCard = document.createElement('div');
                vmCard.className = 'vm-card';
                
                const statusClass = vm.isCompleted() ? 'completed' : 
                    (vm.status === 'running' ? 'running' : 'idle');
                const statusText = vm.isCompleted() ? 'Completed' : 
                    (vm.status === 'running' ? 'Running' : 'Idle');
                
                let processesHTML = '';
                vm.processes.forEach(process => {
                    const processStatusClass = process.status;
                    const processStatusText = process.status.charAt(0).toUpperCase() + process.status.slice(1);
                    
                    processesHTML += `
                        <div class="process-item ${process.status === 'running' ? 'active' : ''} ${process.status === 'completed' ? 'completed' : ''}">
                            <div class="process-header">
                                <span class="process-name">
                                    ${process.status === 'running' ? '▶️' : (process.status === 'completed' ? '✅' : '⏳')}
                                    ${process.name}
                                </span>
                                <span class="process-status ${processStatusClass}">${processStatusText}</span>
                            </div>
                            <div class="process-progress">
                                <div class="process-progress-fill" style="width: ${process.progress}%"></div>
                            </div>
                            <div class="process-info">
                                <span>Remaining: ${Math.max(0, process.remainingTime)}ms</span>
                                <span>Total: ${process.burstTime}ms</span>
                            </div>
                        </div>
                    `;
                });
                
                vmCard.innerHTML = `
                    <div class="vm-header">
                        <div class="vm-name">
                            <div class="vm-icon">🖥️</div>
                            ${vm.name}
                        </div>
                        <span class="vm-status ${statusClass}">${statusText}</span>
                    </div>
                    <div class="process-list">
                        ${processesHTML}
                    </div>
                `;
                
                grid.appendChild(vmCard);
            });
        }

        /**
         * Update statistics display
         */
        function updateStats() {
            const totalVMs = virtualMachines.length;
            const activeProcesses = virtualMachines.reduce((acc, vm) => 
                acc + vm.processes.filter(p => p.status !== 'completed').length, 0);
            const completedProcesses = virtualMachines.reduce((acc, vm) => 
                acc + vm.processes.filter(p => p.status === 'completed').length, 0);
            const totalProcesses = virtualMachines.reduce((acc, vm) => 
                acc + vm.processes.length, 0);
            
            document.getElementById('totalVms').textContent = totalVMs;
            document.getElementById('activeProcesses').textContent = activeProcesses;
            document.getElementById('completedProcesses').textContent = completedProcesses;
            document.getElementById('simTime').textContent = `${(simulationTime / 1000).toFixed(1)}s`;
            
            // Update CPU meter
            const cpuUsage = totalProcesses > 0 ? 
                ((totalProcesses - completedProcesses) / totalProcesses) * 100 : 0;
            document.getElementById('cpuFill').style.width = `${cpuUsage}%`;
            document.getElementById('cpuPercentage').textContent = `${Math.round(cpuUsage)}%`;
        }

        // ============================================
        // WINDOW RESIZE HANDLER
        // ============================================
        window.addEventListener('resize', () => {
            if (chartInstance) {
                drawChart();
            }
        });

        // ============================================
        // INITIALIZE ON LOAD
        // ============================================
        document.addEventListener('DOMContentLoaded', () => {
            // Initialize chart if on simulator page
            if (document.getElementById('simulator').classList.contains('active')) {
                initChart();
            }
        });
    </script>
</body>
</html>
