    <!DOCTYPE html>
    <html lang="es">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
        <title>Carnet Digital Interactivo - SegurApp Recorridos</title>
        <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
        <!-- html2canvas para la descarga del carnet como imagen -->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
        <!-- Leaflet CSS y JS para el mapa interactivo de rutas -->
        <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
        <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
        <style>
            :root {
                --bg-principal: #08080a;
                --bg-tarjeta: #000000;
                --bg-input: #12151c;
                --acero-claro: #a6b4c9;
                --acero-oscuro: #3a4454;
                --dorado-brillante: #d4af37;
                --dorado-base: #b88628;
                --dorado-oscuro: #997a15;
                --glow-dorado: rgba(212, 175, 55, 0.35);
                --verde-verificado: #00ff88;
                --glow-verde: rgba(0, 255, 136, 0.6);
                
                --texto-principal: #ffffff;
                --texto-secundario: #e1e7f0;
                --borde-sutil: rgba(212, 175, 55, 0.4);
                --fuente: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            }

            * {
                box-sizing: border-box;
                margin: 0;
                padding: 0;
                -webkit-tap-highlight-color: transparent;
                font-family: var(--fuente);
            }

            body {
                background-color: var(--bg-principal);
                color: var(--texto-principal);
                min-height: 100vh;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                padding: 20px;
                position: relative;
                overflow-x: hidden;
            }

            .video-fondo {
                position: fixed;
                top: 0;
                left: 0;
                width: 100vw;
                height: 100vh;
                object-fit: cover;
                z-index: -2;
            }

            body::before {
                content: '';
                position: fixed;
                top: 0; left: 0; width: 100%; height: 100%;
                background: radial-gradient(circle at center, rgba(212, 175, 55, 0.08) 0%, rgba(8, 8, 10, 0.98) 80%);
                z-index: -1;
                pointer-events: none;
            }

            /* --- SISTEMA DE ALERTAS Y LETREROS INTERACTIVOS --- */
            #toastContainer {
                position: fixed;
                top: 20px;
                right: 20px;
                z-index: 99999;
                display: flex;
                flex-direction: column;
                gap: 10px;
                max-width: 360px;
                width: 100%;
                pointer-events: none;
            }

            .custom-toast {
                background: rgba(12, 15, 22, 0.95);
                border: 2px solid var(--dorado-brillante);
                border-radius: 14px;
                padding: 14px 18px;
                color: #fff;
                box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px var(--glow-dorado);
                display: flex;
                align-items: flex-start;
                gap: 12px;
                pointer-events: auto;
                transform: translateX(120%);
                transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275), opacity 0.3s ease;
                opacity: 0;
                backdrop-filter: blur(10px);
            }

            .custom-toast.show {
                transform: translateX(0);
                opacity: 1;
            }

            .custom-toast.success {
                border-color: var(--verde-verificado);
                box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px var(--glow-verde);
            }

            .custom-toast.error {
                border-color: #ff4444;
                box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px rgba(255, 68, 68, 0.4);
            }

            .toast-icon {
                font-size: 24px;
                color: var(--dorado-brillante);
                flex-shrink: 0;
            }

            .custom-toast.success .toast-icon {
                color: var(--verde-verificado);
            }

            .custom-toast.error .toast-icon {
                color: #ff4444;
            }

            .toast-content {
                flex: 1;
                display: flex;
                flex-direction: column;
                gap: 2px;
            }

            .toast-title {
                font-size: 0.9rem;
                font-weight: 800;
                text-transform: uppercase;
                letter-spacing: 0.5px;
                color: #fff;
            }

            .toast-msg {
                font-size: 0.82rem;
                color: var(--texto-secundario);
                line-height: 1.4;
            }

            .toast-close {
                background: none;
                border: none;
                color: var(--texto-secundario);
                font-size: 18px;
                cursor: pointer;
                padding: 0;
                line-height: 1;
            }

            .toast-close:hover {
                color: #fff;
            }

            /* --- MODAL DE ALERTA INTERACTIVO (CUSTOM PROMPT/CONFIRM) --- */
            .interactive-alert-overlay {
                position: fixed;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: rgba(0, 0, 0, 0.85);
                backdrop-filter: blur(8px);
                display: flex;
                align-items: center;
                justify-content: center;
                z-index: 100000;
                opacity: 0;
                pointer-events: none;
                transition: opacity 0.3s ease;
                padding: 20px;
            }

            .interactive-alert-overlay.active {
                opacity: 1;
                pointer-events: auto;
            }

            .interactive-alert-box {
                background: #000000;
                border: 2px solid var(--dorado-brillante);
                border-radius: 20px;
                width: 100%;
                max-width: 400px;
                padding: 24px;
                box-shadow: 0 15px 35px rgba(0,0,0,0.9), 0 0 30px var(--glow-dorado);
                display: flex;
                flex-direction: column;
                gap: 16px;
                text-align: center;
                transform: scale(0.8);
                transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            }

            .interactive-alert-overlay.active .interactive-alert-box {
                transform: scale(1);
            }

            .interactive-alert-icon {
                font-size: 48px;
                color: var(--dorado-brillante);
                margin: 0 auto;
                text-shadow: 0 0 15px var(--glow-dorado);
            }

            .interactive-alert-title {
                font-size: 1.15rem;
                font-weight: 900;
                color: #fff;
                text-transform: uppercase;
                letter-spacing: 1px;
            }

            .interactive-alert-msg {
                font-size: 0.92rem;
                color: var(--texto-secundario);
                line-height: 1.5;
            }

            .interactive-alert-input {
                background: var(--bg-input);
                border: 1px solid var(--borde-sutil);
                color: #fff;
                padding: 12px 14px;
                border-radius: 10px;
                font-size: 0.95rem;
                outline: none;
                width: 100%;
                text-align: center;
            }

            .interactive-alert-input:focus {
                border-color: var(--dorado-brillante);
                box-shadow: 0 0 10px var(--glow-dorado);
            }

            .interactive-alert-buttons {
                display: flex;
                gap: 10px;
                margin-top: 4px;
            }

            .interactive-alert-btn {
                flex: 1;
                padding: 12px;
                border-radius: 10px;
                font-weight: 800;
                font-size: 0.9rem;
                cursor: pointer;
                text-transform: uppercase;
                border: none;
                transition: all 0.2s ease;
            }

            .interactive-alert-btn.primary {
                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                box-shadow: 0 0 15px var(--glow-dorado);
            }

            .interactive-alert-btn.secondary {
                background: rgba(255, 255, 255, 0.1);
                color: #fff;
                border: 1px solid var(--borde-sutil);
            }

            .interactive-alert-btn.danger {
                background: linear-gradient(135deg, #ff4444 0%, #aa0000 100%);
                color: #fff;
                box-shadow: 0 0 15px rgba(255, 68, 68, 0.4);
            }

            /* --- BARRA USUARIO SUPERIOR --- */
            .user-top-bar {
                width: 100%;
                max-width: 380px;
                display: flex;
                justify-content: space-between;
                align-items: center;
                background: rgba(18, 21, 28, 0.9);
                border: 1px solid var(--borde-sutil);
                padding: 8px 14px;
                border-radius: 30px;
                margin-bottom: 12px;
                box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            }

            .user-info-pill {
                display: flex;
                align-items: center;
                gap: 10px;
                font-size: 0.85rem;
                color: #fff;
                font-weight: 700;
            }

            .user-avatar-mini {
                width: 32px;
                height: 32px;
                border-radius: 50%;
                border: 1.5px solid var(--dorado-brillante);
                object-fit: cover;
                background-color: #000;
            }

            .top-bar-buttons {
                display: flex;
                gap: 6px;
            }

            .btn-auth-trigger {
                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                border: none;
                padding: 6px 12px;
                border-radius: 20px;
                font-size: 0.78rem;
                font-weight: 800;
                cursor: pointer;
                display: flex;
                align-items: center;
                gap: 4px;
                text-transform: uppercase;
            }

            .btn-edit-trigger {
                background: rgba(212, 175, 55, 0.15);
                color: var(--dorado-brillante);
                border: 1px solid var(--dorado-brillante);
                padding: 6px 10px;
                border-radius: 20px;
                font-size: 0.78rem;
                font-weight: 800;
                cursor: pointer;
                display: flex;
                align-items: center;
                gap: 4px;
                text-transform: uppercase;
            }

            /* --- INDICADOR DE GIRAR --- */
            .hint-girar {
                font-size: 0.95rem;
                color: #ffffff;
                text-transform: uppercase;
                letter-spacing: 1.2px;
                font-weight: 900;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 8px;
                margin-bottom: 16px;
                padding: 10px 20px;
                background: linear-gradient(135deg, rgba(22, 25, 32, 0.95) 0%, rgba(10, 12, 16, 0.95) 100%);
                border: 2px solid var(--dorado-brillante);
                border-radius: 30px;
                box-shadow: 0 0 15px rgba(255, 215, 0, 0.4), inset 0 0 10px rgba(255, 215, 0, 0.1);
                animation: pulseHint 1.8s infinite ease-in-out;
                cursor: pointer;
                user-select: none;
            }

            .hint-girar .icono-girar {
                font-size: 22px;
                color: var(--dorado-brillante);
                animation: rotarIcono 2.5s infinite linear;
            }

            @keyframes pulseHint {
                0%, 100% {
                    transform: scale(1);
                    box-shadow: 0 0 12px rgba(255, 215, 0, 0.4), 0 0 20px rgba(212, 175, 55, 0.2);
                }
                50% {
                    transform: scale(1.05);
                    box-shadow: 0 0 22px rgba(255, 215, 0, 0.8), 0 0 35px rgba(212, 175, 55, 0.5);
                    border-color: #ffe600;
                }
            }

            @keyframes rotarIcono {
                0% { transform: rotate(0deg); }
                100% { transform: rotate(360deg); }
            }

            /* --- CONTENEDOR FLIP 3D --- */
            .escena-carnet {
                width: 100%;
                max-width: 380px;
                height: 680px;
                perspective: 1200px;
                margin-bottom: 20px;
                cursor: pointer;
            }

            .carnet-inner {
                width: 100%;
                height: 100%;
                position: relative;
                transform-style: preserve-3d;
                transition: transform 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            }

            .escena-carnet.flipped .carnet-inner {
                transform: rotateY(180deg);
            }

            .carnet-cara {
                position: absolute;
                width: 100%;
                height: 100%;
                backface-visibility: hidden;
                -webkit-backface-visibility: hidden;
                background: var(--bg-tarjeta);
                border: 2px solid var(--dorado-brillante);
                border-radius: 20px;
                padding: 14px 18px;
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                overflow: hidden;
                box-shadow: 0 15px 35px rgba(0, 0, 0, 0.9), 0 0 20px var(--glow-dorado);
            }

            .carnet-cara.reverso {
                transform: rotateY(180deg);
            }

            @keyframes barridoMetalico {
                0% { left: -150%; }
                50% { left: -150%; }
                100% { left: 150%; }
            }

            .carnet-cara::after {
                content: '';
                position: absolute;
                top: 0;
                width: 100%;
                height: 100%;
                background: linear-gradient(
                    90deg,
                    rgba(255, 215, 0, 0) 0%,
                    rgba(255, 215, 0, 0.03) 20%,
                    rgba(255, 215, 0, 0.15) 50%,
                    rgba(255, 215, 0, 0.03) 80%,
                    rgba(255, 215, 0, 0) 100%
                );
                transform: skewX(-25deg);
                animation: barridoMetalico 6s infinite ease-in-out;
                pointer-events: none;
            }

            .perforacion-lanyard {
                width: 50px;
                height: 10px;
                background: #111;
                border: 1px solid var(--borde-sutil);
                border-radius: 10px;
                margin: 0 auto 4px auto;
                box-shadow: inset 0 2px 4px rgba(0,0,0,0.8);
            }

            .carnet-header {
                text-align: center;
                border-bottom: 1px solid var(--borde-sutil);
                padding-bottom: 4px;
                margin-bottom: 2px;
                display: flex;
                flex-direction: column;
                align-items: center;
            }

            .carnet-header img.logo {
                max-width: 50px;
                height: auto;
                margin-bottom: 2px;
            }

            .frontal .carnet-header .sub-brand, .reverso .carnet-header .sub-brand {
                font-size: 0.84rem;
                color: var(--dorado-brillante);
                letter-spacing: 1.5px;
                text-transform: uppercase;
                font-weight: 800;
                text-shadow: 0 0 8px rgba(255, 215, 0, 0.5);
            }

            .badge-rol {
                display: inline-block;
                background: #000000;
                color: var(--dorado-brillante);
                border: 1.5px solid var(--dorado-brillante);
                font-size: 0.84rem;
                font-weight: 900;
                padding: 3px 12px;
                border-radius: 20px;
                text-transform: uppercase;
                letter-spacing: 1px;
                margin-top: 2px;
                box-shadow: 0 0 10px var(--glow-dorado);
            }

            .carnet-body {
                display: flex;
                flex-direction: column;
                align-items: center;
                text-align: center;
            }

            .foto-marco {
                position: relative;
                margin-top: 4px;
                margin-bottom: 2px;
            }

            .foto-conductor {
                width: 170px;
                height: 170px;
                border-radius: 50%;
                object-fit: cover;
                border: 3px solid var(--dorado-brillante);
                box-shadow: 0 0 22px var(--glow-dorado);
                background-color: #000;
            }

            .badge-verificado-icono {
                position: absolute;
                bottom: 15px;
                right: 11px;
                background: #1e501c;
                color: var(--verde-verificado);
                border-radius: 50%;
                width: 30px;
                height: 30px;
                display: flex;
                align-items: center;
                justify-content: center;
        <source src="img/Rediseño de página moderno.mp4" type="video/mp4">
                border: 2px solid var(--verde-verificado);
                box-shadow: 0 0 15px var(--glow-verde);
                z-index: 10;
            }

            .badge-verificado-icono::after {
                content: '';
                position: absolute;
                width: 100%;
                height: 100%;
                border-radius: 50%;
                border: 2px solid var(--verde-verificado);
                opacity: 0;
                animation: pulsoVerde 2s infinite;
                box-sizing: border-box;
            }

            @keyframes pulsoVerde {
                0% { transform: scale(1); opacity: 1; }
                100% { transform: scale(1.5); opacity: 0; }
            }

            .frontal .nombre-conductor {
                font-size: 1.26rem;
                font-weight: 800;
                color: #ffffff;
                margin-top: 2px;
                margin-bottom: 2px;
            }

            .frontal .cargo-conductor {
                font-size: 0.88rem;
                color: var(--texto-secundario);
                font-weight: 500;
                margin-bottom: 8px;
            }

            .datos-grid {
                width: 100%;
                background: #000000;
                border-radius: 12px;
                padding: 8px 10px;
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 6px;
                text-align: left;
                border: 1px solid var(--borde-sutil);
                margin-bottom: 8px;
            }

            .dato-item {
                display: flex;
                flex-direction: column;
            }

            .dato-item.full-width {
                grid-column: span 2;
            }

            .dato-label {
                font-size: 0.75rem;
                color: var(--texto-secundario);
                text-transform: uppercase;
                font-weight: 700;
                letter-spacing: 0.3px;
            }

            .dato-val {
                font-size: 0.90rem;
                color: #ffffff;
                font-weight: 700;
            }

            .dato-val.destacado {
                color: var(--dorado-brillante);
                text-shadow: 0 0 5px rgba(255, 215, 0, 0.3);
            }

            .qr-destacado-container {
                display: flex;
                justify-content: center;
                align-items: center;
                margin: 4px 0 8px 0;
            }

            .qr-code-lg {
                width: 160px;
                height: 160px;
                background: #ffffff;
                padding: 8px;
                border-radius: 14px;
                box-shadow: 0 0 20px rgba(212, 175, 55, 0.4), 0 4px 12px rgba(0,0,0,0.5);
                border: 3px solid var(--dorado-brillante);
            }

            .qr-code-lg img {
                width: 100%;
                height: 100%;
                display: block;
                object-fit: contain;
            }

            .carnet-footer {
                display: flex;
                align-items: center;
                justify-content: space-around;
                width: 100%;
                background: #000000;
                padding: 8px 12px;
                border-radius: 12px;
                border: 1px dashed var(--borde-sutil);
            }

            .info-qr {
                text-align: center;
                width: 100%;
                display: flex;
                justify-content: space-around;
                align-items: center;
            }

            .info-qr .id-codigo {
                font-family: monospace;
                font-size: 0.90rem;
                color: var(--texto-principal);
                font-weight: 700;
            }

            .info-qr .estado {
                font-size: 0.80rem;
                color: var(--verde-verificado);
                font-weight: 800;
                text-transform: uppercase;
                display: flex;
                align-items: center;
                gap: 4px;
                text-shadow: 0 0 5px var(--glow-verde);
            }

            .rating-box-reverso {
                width: 100%;
                background: #000000;
                border: 1px solid var(--borde-sutil);
                border-radius: 12px;
                padding: 10px;
                margin: 10px 0;
                display: flex;
                flex-direction: column;
                align-items: center;
            }

            .rating-title {
                font-size: 0.78rem;
                color: var(--dorado-brillante);
                text-transform: uppercase;
                font-weight: 800;
                letter-spacing: 0.5px;
                margin-bottom: 4px;
            }

            .rating-promedio {
                font-size: 0.98rem;
                font-weight: 800;
                color: #ffffff;
                display: flex;
                align-items: center;
                gap: 4px;
            }

            .stars-container {
                display: flex;
                gap: 6px;
                margin: 6px 0;
            }

            .star-icon {
                font-size: 24px;
                color: #5a6270;
                cursor: pointer;
                transition: all 0.2s ease;
            }

            .star-icon.active, .star-icon:hover {
                color: #ffd700;
                text-shadow: 0 0 10px rgba(255, 215, 0, 0.7);
            }

            .voto-mensaje {
                font-size: 0.78rem;
                color: var(--verde-verificado);
                margin-top: 2px;
                font-weight: 700;
            }

            .reverso-info {
                font-size: 0.85rem;
                color: var(--texto-secundario);
                line-height: 1.4;
                text-align: center;
            }

            .acciones-bar {
                width: 100%;
                max-width: 380px;
                display: flex;
                flex-direction: column;
                gap: 12px;
            }

            .btn-grupo-principal {
                display: flex;
                gap: 10px;
            }

            .btn-accion {
                flex: 1;
                background: #111318;
                border: 1px solid var(--borde-sutil);
                color: #fff;
                padding: 14px;
                border-radius: 14px;
                font-size: 0.95rem;
                font-weight: 700;
                cursor: pointer;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 8px;
                transition: all 0.2s ease;
                text-decoration: none;
                box-shadow: 0 4px 12px rgba(0,0,0,0.5);
            }

            .btn-accion:active {
                transform: scale(0.96);
            }

            .btn-accion.solicitar-wasap {
                background: linear-gradient(135deg, #00ff88 0%, #25d366 50%, #056232 100%);
                color: #000000;
                border: 2px solid #00ff88;
                font-size: 1.15rem;
                font-weight: 900;
                text-transform: uppercase;
                letter-spacing: 0.8px;
                padding: 18px 24px;
                border-radius: 20px;
                box-shadow: 0 0 25px rgba(0, 255, 136, 0.7), inset 0 2px 4px rgba(255,255,255,0.8);
                animation: superPulso 1.8s infinite ease-in-out;
                cursor: pointer;
                width: 100%;
            }

            .btn-accion.guardar-contacto {
                                 padding: 8px 5px;

                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                border: none;
                box-shadow: 0 4px 15px var(--glow-dorado);
            }

            .btn-accion.quienes-somos {
                                 padding: 8px 5px;

                background: rgba(22, 25, 32, 0.9);
                border: 1px solid var(--dorado-brillante);
                color: var(--dorado-brillante);
                box-shadow: 0 0 10px rgba(212, 175, 55, 0.2);
            }

            .btn-accion.terminos-condiciones {
                 padding: 8px 4px;
                background: rgba(22, 25, 32, 0.9);
                border: 1px solid #ff4444;
                color: #ff6666;
                box-shadow: 0 0 10px rgba(255, 68, 68, 0.2);
            }

            .btn-accion.descargar-carnet {
                background: rgba(22, 25, 32, 0.9);
                border: 1px solid var(--acero-claro);
                color: #ffffff;
                box-shadow: 0 0 10px rgba(166, 180, 201, 0.2);
            }

            .btn-accion.panel-conductor {
                background: linear-gradient(135deg, #12151c 0%, #1f2633 100%);
                border: 2px solid var(--dorado-brillante);
                color: var(--dorado-brillante);
                box-shadow: 0 0 15px rgba(212, 175, 55, 0.3);
            }

            /* --- MODALES --- */
            .modal-overlay {
                position: fixed;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: rgba(0, 0, 0, 0.85);
                backdrop-filter: blur(8px);
                display: flex;
                align-items: center;
                justify-content: center;
                z-index: 1000;
                opacity: 0;
                pointer-events: none;
                transition: opacity 0.3s ease;
                padding: 20px;
            }

            .modal-overlay.active {
                opacity: 1;
                pointer-events: auto;
            }

            .modal-contenido {
                background: #000000;
                border: 1px solid var(--borde-sutil);
                border-radius: 20px;
                width: 100%;
                max-width: 480px;
                max-height: 88vh;
                overflow-y: auto;
                padding: 24px;
                box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 25px var(--glow-dorado);
                position: relative;
                transform: translateY(20px);
                transition: transform 0.3s ease;
            }

            .modal-overlay.active .modal-contenido {
                transform: translateY(0);
            }

            .modal-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                border-bottom: 1px solid var(--borde-sutil);
                padding-bottom: 12px;
                margin-bottom: 16px;
            }

            .modal-titulo {
                font-size: 1.15rem;
                font-weight: 800;
                color: var(--dorado-brillante);
                text-transform: uppercase;
                letter-spacing: 1px;
                display: flex;
                align-items: center;
                gap: 8px;
            }

            .btn-cerrar {
                background: none;
                border: none;
                color: var(--texto-secundario);
                font-size: 26px;
                cursor: pointer;
                display: flex;
                align-items: center;
                justify-content: center;
            }

            .modal-body {
                font-size: 0.92rem;
                color: var(--texto-secundario);
                line-height: 1.6;
                display: flex;
                flex-direction: column;
                gap: 14px;
            }

            /* --- TÍTULOS DE TÉRMINOS Y CONDICIONES (CAMBIADOS DE COLOR E IDENTIFICADOS) --- */
            .modal-body h4, .terminos-titulo-seccion {
                color: var(--dorado-brillante) !important;
                font-size: 1.02rem;
                margin-top: 6px;
                display: flex;
                align-items: center;
                gap: 6px;
                border-bottom: 1px solid rgba(212, 175, 55, 0.3);
                padding-bottom: 4px;
                text-shadow: 0 0 8px rgba(212, 175, 55, 0.4);
            }

            .caracteristica-box {
                background: #000000;
                border: 1px solid var(--borde-sutil);
                border-radius: 10px;
                padding: 10px;
                margin-top: 4px;
            }

            /* --- FORMULARIOS / AUTENTICACIÓN --- */
            .auth-tabs {
                display: flex;
                gap: 6px;
                margin-bottom: 16px;
                border-bottom: 1px solid var(--borde-sutil);
                padding-bottom: 10px;
            }

            .auth-tab-btn {
                flex: 1;
                background: transparent;
                border: none;
                color: var(--texto-secundario);
                padding: 8px 4px;
                font-weight: 700;
                font-size: 0.78rem;
                cursor: pointer;
                border-radius: 8px;
                transition: all 0.2s ease;
                text-align: center;
            }

            .auth-tab-btn.active {
                background: var(--dorado-brillante);
                color: #000;
            }

            .form-group {
                display: flex;
                flex-direction: column;
                gap: 6px;
                text-align: left;
                margin-bottom: 10px;
            }

            .form-group label {
                font-size: 0.8rem;
                font-weight: 700;
                color: var(--dorado-brillante);
                text-transform: uppercase;
            }

            .form-group input, .form-group select {
                background: var(--bg-input);
                border: 1px solid var(--borde-sutil);
                color: #fff;
                padding: 10px 3px;
                border-radius: 10px;
                font-size: 0.9rem;
                outline: none;
            }

            .form-group input:focus, .form-group select:focus {
                border-color: var(--dorado-brillante);
                box-shadow: 0 0 8px var(--glow-dorado);
            }

            .avatar-upload-preview {
                display: flex;
                align-items: center;
                gap: 12px;
                margin-bottom: 10px;
            }

            .preview-circle {
                width: 60px;
                height: 60px;
                border-radius: 50%;
                border: 2px solid var(--dorado-brillante);
                object-fit: cover;
                background: #111;
            }

            .btn-submit-auth {
                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                border: none;
                padding: 12px;
                border-radius: 10px;
                font-weight: 800;
                font-size: 0.95rem;
                cursor: pointer;
                text-transform: uppercase;
                letter-spacing: 0.5px;
                margin-top: 8px;
                width: 100%;
                box-shadow: 0 0 10px var(--glow-dorado);
            }

            .link-forgot-pass {
                font-size: 0.8rem;
                color: var(--dorado-brillante);
                text-decoration: underline;
                cursor: pointer;
                text-align: right;
                display: block;
                margin-top: 4px;
            }

            /* --- PANEL DE FINANZAS / CUENTAS DEL CONDUCTOR --- */
            .finanzas-dashboard {
                display: flex;
                flex-direction: column;
                gap: 16px;
            }

            .resumen-cards-grid {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 10px;
            }

            .card-metrica {
                background: #0f1218;
                border: 1px solid var(--borde-sutil);
                border-radius: 12px;
                padding: 12px;
                display: flex;
                flex-direction: column;
                gap: 4px;
            }

            .card-metrica .titulo-metrica {
                font-size: 0.72rem;
                color: var(--texto-secundario);
                text-transform: uppercase;
                font-weight: 700;
            }

            .card-metrica .valor-metrica {
                font-size: 1.1rem;
                font-weight: 900;
                color: #fff;
            }

            .card-metrica .valor-metrica.ingreso {
                color: var(--verde-verificado);
            }

            .card-metrica .valor-metrica.gasto {
                color: #ff6666;
            }

            .card-metrica .valor-metrica.balance {
                color: var(--dorado-brillante);
            }

            .seccion-transacciones {
                background: #0f1218;
                border: 1px solid var(--borde-sutil);
                border-radius: 12px;
                padding: 12px;
                max-height: 220px;
                overflow-y: auto;
            }

            .transaccion-item {
                display: flex;
                justify-content: space-between;
                align-items: center;
                padding: 8px 0;
                border-bottom: 1px solid rgba(255,255,255,0.05);
                font-size: 0.82rem;
            }

            .transaccion-item:last-child {
                border-bottom: none;
            }

            .transaccion-info {
                display: flex;
                flex-direction: column;
            }

            .transaccion-desc {
                font-weight: 700;
                color: #fff;
            }

            .transaccion-fecha {
                font-size: 0.68rem;
                color: var(--texto-secundario);
            }

            .transaccion-monto {
                font-weight: 800;
            }

            .transaccion-monto.ingreso {
                color: var(--verde-verificado);
            }

            .transaccion-monto.gasto {
                color: #ff6666;
            }

            .acciones-finanzas {
                display: flex;
                gap: 8px;
            }

            .btn-exportar-csv {
                background: rgba(0, 255, 136, 0.15);
                color: var(--verde-verificado);
                border: 1px solid var(--verde-verificado);
                padding: 10px;
                border-radius: 10px;
                font-size: 0.8rem;
                font-weight: 800;
                cursor: pointer;
                flex: 1;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 6px;
            }

            .btn-exportar-img {
                background: rgba(212, 175, 55, 0.15);
                color: var(--dorado-brillante);
                border: 1px solid var(--dorado-brillante);
                padding: 10px;
                border-radius: 10px;
                font-size: 0.8rem;
                font-weight: 800;
                cursor: pointer;
                flex: 1;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 6px;
            }

            /* --- MAPA Y RASTREADOR DE RECORRIDOS (MODERNO E INTERACTIVO) --- */
            .seccion-mapa-recorrido {
                background: #0f1218;
                border: 1px solid var(--borde-sutil);
                border-radius: 12px;
                padding: 12px;
                display: flex;
                flex-direction: column;
                gap: 10px;
            }

            #mapContainer {
                width: 100%;
                height: 280px;
                border-radius: 10px;
                border: 1px solid var(--borde-sutil);
                background: #000;
                z-index: 1;
            }

            .map-search-bar {
                display: flex;
                gap: 6px;
            }

            .map-search-bar input {
                flex: 1;
                background: var(--bg-input);
                border: 1px solid var(--borde-sutil);
                color: #fff;
                padding: 8px 10px;
                border-radius: 8px;
                font-size: 0.85rem;
                outline: none;
            }

            .map-search-bar input:focus {
                border-color: var(--dorado-brillante);
                box-shadow: 0 0 6px var(--glow-dorado);
            }

            .btn-map-action {
                background: rgba(212, 175, 55, 0.2);
                color: var(--dorado-brillante);
                border: 1px solid var(--dorado-brillante);
                padding: 8px 12px;
                border-radius: 8px;
                font-size: 0.8rem;
                font-weight: 700;
                cursor: pointer;
                display: flex;
                align-items: center;
                gap: 4px;
            }

            .marcadores-guardados-box {
                background: #000;
                border: 1px solid var(--borde-sutil);
                border-radius: 8px;
                padding: 8px;
                max-height: 100px;
                overflow-y: auto;
                font-size: 0.8rem;
            }

            .marcador-pill {
                display: flex;
                justify-content: space-between;
                align-items: center;
                padding: 4px 6px;
                border-bottom: 1px solid rgba(255,255,255,0.05);
            }

            .marcador-pill:last-child {
                border-bottom: none;
            }

            .marcador-nombre {
                color: #fff;
                font-weight: 600;
                cursor: pointer;
            }

            .marcador-nombre:hover {
                color: var(--dorado-brillante);
                text-decoration: underline;
            }

            .controles-recorrido {
                display: flex;
                gap: 8px;
            }

            .btn-control-mapa {
                flex: 1;
                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                border: none;
                padding: 10px;
                border-radius: 8px;
                font-weight: 800;
                font-size: 0.82rem;
                cursor: pointer;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 6px;
                text-transform: uppercase;
            }

            .btn-control-mapa.detener {
                background: linear-gradient(135deg, #ff4444 0%, #aa0000 100%);
                color: #fff;
            }

            /* PROTOCOLO DE SEGURIDAD */
            .alerta-seguridad-box {
                position: relative;
                background: linear-gradient(145deg, rgba(35, 10, 10, 0.95) 0%, rgba(15, 5, 5, 0.98) 100%);
                border: 2px solid #ff3333;
                border-radius: 16px;
                padding: 16px;
                color: #ffffff;
                font-size: 0.9rem;
                box-shadow: 0 0 20px rgba(255, 51, 51, 0.35);
                margin-bottom: 6px;
            }

            .alerta-header {
                display: flex;
                align-items: center;
                justify-content: space-between;
                border-bottom: 1px solid rgba(255, 85, 85, 0.3);
                padding-bottom: 8px;
                margin-bottom: 10px;
            }

            .alerta-titulo-text {
                display: flex;
                align-items: center;
                gap: 8px;
                color: #ff5555;
                font-weight: 900;
                font-size: 0.95rem;
                letter-spacing: 0.8px;
                text-transform: uppercase;
            }

            .badge-live-seguridad {
                background: rgba(255, 51, 51, 0.2);
                color: #ff6666;
                border: 1px solid #ff4444;
                font-size: 0.68rem;
                font-weight: 800;
                padding: 2px 8px;
                border-radius: 12px;
                display: flex;
                align-items: center;
                gap: 4px;
                text-transform: uppercase;
            }

            .dot-live {
                width: 7px;
                height: 7px;
                background-color: #ff3333;
                border-radius: 50%;
                box-shadow: 0 0 8px #ff3333;
                animation: pulsoRedLive 1.2s infinite ease-in-out;
            }

            @keyframes pulsoRedLive {
                0%, 100% { opacity: 1; transform: scale(1); }
                50% { opacity: 0.3; transform: scale(1.4); }
            }

            .alerta-puntos-claves {
                display: grid;
                grid-template-columns: 1fr;
                gap: 6px;
            }

            .punto-seguridad-item {
                background: rgba(0, 0, 0, 0.6);
                border: 1px solid rgba(255, 85, 85, 0.25);
                border-radius: 8px;
                padding: 6px 10px;
                display: flex;
                align-items: center;
                gap: 8px;
                font-size: 0.82rem;
                color: #ffffff;
                font-weight: 600;
            }

            .btn-aceptar-terminos {
                width: 100%;
                background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
                color: #000;
                border: none;
                padding: 14px;
                border-radius: 12px;
                font-weight: 800;
                font-size: 1rem;
                cursor: pointer;
                text-transform: uppercase;
                letter-spacing: 1px;
                box-shadow: 0 0 15px var(--glow-dorado);
                margin-top: 10px;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 8px;
            }
        </style>
    </head> 
    <body>

        <!-- CONTENEDOR DE TOASTS / LETREROS INTERACTIVOS -->
        <div id="toastContainer"></div>

        <!-- MODAL DE ALERTA INTERACTIVO (REEMPLAZO DE ALERT / PROMPT / CONFIRM) -->
        <div class="interactive-alert-overlay" id="interactiveAlertModal">
            <div class="interactive-alert-box">
                <span class="material-icons interactive-alert-icon" id="interactiveAlertIcon">notifications_active</span>
                <div class="interactive-alert-title" id="interactiveAlertTitle">Aviso SegurApp</div>
                <div class="interactive-alert-msg" id="interactiveAlertMsg">Mensaje de prueba interactivo.</div>
                <div id="interactiveAlertInputContainer" style="display: none;">
                    <input type="text" id="interactiveAlertInputField" class="interactive-alert-input" placeholder="Escribe aquí...">
                </div>
                <div class="interactive-alert-buttons" id="interactiveAlertButtons">
                    <button class="interactive-alert-btn primary" id="interactiveAlertBtnOk" onclick="cerrarAlertaInteractiva(true)">Aceptar</button>
                    <button class="interactive-alert-btn secondary" id="interactiveAlertBtnCancel" onclick="cerrarAlertaInteractiva(false)" style="display: none;">Cancelar</button>
                </div>
            </div>
        </div>

        <video class="video-fondo" autoplay muted loop playsinline>
            <source src="img/Rediseño de página moderno.mp4" type="video/mp4">
            Tu navegador no soporta video de fondo.
        </video>

        <!-- BARRA SUPERIOR DE SESIÓN -->
        <div class="user-top-bar" id="userTopBar">
            <div class="user-info-pill">
                <img src="https://ui-avatars.com/api/?name=Usuario&background=333&color=fff" id="barUserAvatar" class="user-avatar-mini" alt="Avatar">
                <span id="barUserName">Invitado</span>
            </div>
            <div class="top-bar-buttons">
                <button class="btn-edit-trigger" id="btnEditUser" style="display: none;" onclick="abrirEditModal()">
                    <span class="material-icons" style="font-size: 15px;">edit</span> <span>Editar</span>
                </button>
                <button class="btn-auth-trigger" id="btnAuthTrigger" onclick="abrirAuthModal()">
                    <span class="material-icons" style="font-size: 16px;">account_circle</span> <span id="lblBtnAuth">Ingresar</span>
                </button>
            </div>
        </div>

        <!-- BOTÓN VISIBLE PARA VOLTEAR -->
        <div class="hint-girar" onclick="voltearCarnet()">
            <span class="material-icons icono-girar">autorenew</span>
            Toca el carnet para voltear
        </div>

        <!-- ESCENA 3D DEL CARNET -->
        <div class="escena-carnet" id="escenaCarnet">
            <div class="carnet-inner">
                
                <!-- CARA FRONTAL -->
                <div class="carnet-cara frontal" onclick="voltearCarnet()">
                    <div class="perforacion-lanyard"></div>

                    <div class="carnet-header">
                        <div class="sub-brand">RAPIDOS - confiables - seguros.</div>
                        <span class="badge-rol">Conductor Oficial</span>
                    </div>

                    <div class="carnet-body">
                        <div class="foto-marco">
                            <img src="img/sergio.jpg" onerror="this.src='https://ui-avatars.com/api/?name=Sergio+Tapiero&background=ffd700&color=000&size=200'" alt="Foto Conductor" class="foto-conductor">
                            <div class="badge-verificado-icono">
                                <span class="material-icons" style="font-size: 18px;">check_circle</span>
                            </div>
                        </div>

                        <div class="nombre-conductor">Sergio Alejandro Tapiero Chala</div>
                        <span class="dato-val destacado">1.006.506.890</span>

                        <div class="cargo-conductor">SegurApp Recorridos</div>

                        <div class="datos-grid">
                            <div class="dato-item full-width">
                                <span class="dato-label">Rol Asignado</span>
                                <span class="dato-val destacado">Gerente de Operaciones</span>
                            </div>
                            <div class="dato-item">
                                <span class="dato-label">vehiculo registrado</span>
                                <span class="dato-val destacado">TVS Raider (Negra)</span>
                            </div>
                            <div class="dato-item">
                                <span class="dato-label">Placa Vehículo</span>
                                <span class="dato-val destacado">BWQ 69H</span>
                            </div>
                        </div>

                        <div class="qr-destacado-container">
                            <div class="qr-code-lg">
                                <img src="https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/764642024_122113423269318735_6046867069515873634_n.jpg?stp=dst-jpg_tt6&cstp=mx2048x2048&ctp=s2048x2048&_nc_cat=102&ccb=1-7&_nc_sid=127cfc&_nc_eui2=AeFO-8QfBG5_LEAT30BzbtCF6z-AvoUnjSXrP4C-hSeNJf_nWuP_OjevNjrvaE-H_CrRJSMPuRM65S6iRSHKi95D&_nc_ohc=Y9WxPWXyBMoQ7kNvwGd4Nuc&_nc_oc=Adpt7yrTZ05sVigl3I2IwRC8zxluf15ljbNUSow5nWVRE-mPx2Xc5KtuBwI9d5lrkYM&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=MEPZ2J5WV3Tiaw9CiL09BA&_nc_ss=7b2a8&oh=00_AQEU3jsrs3hDzMugL3JvpqYdfunDq46PqO_LvnghWA7C7Q&oe=6A75B3D1" alt="Código QR WhatsApp">
                            </div>
                        </div>
                    </div>

                    <div class="carnet-footer">
                        <div class="info-qr">
                            <div class="id-codigo">ID: SEG-2026-890</div>
                            <div class="estado">
                                <span class="material-icons" style="font-size: 12px;">circle</span> Activo / Verificado
                            </div>
                        </div>
                    </div> 
                </div>

                <!-- CARA POSTERIOR (REVERSO) -->
                <div class="carnet-cara reverso" onclick="voltearCarnet()">
                    <div class="perforacion-lanyard"></div>
                    
                    <div class="carnet-header">
                        <img src="img/png.png" alt="SegurApp Logo" class="logo" style="max-width: 250px;">
                        <div class="sub-brand">Información y Valoración</div>
                    </div>

                    <div class="reverso-info">
                        <p>Acredita el rol de <strong>Conductor Autorizado</strong> en <strong>SegurApp Recorridos</strong>.</p>
                        
                        <div class="rating-box-reverso" onclick="event.stopPropagation()">
                            <div class="rating-title">Calificación del Conductor</div>
                            <div class="rating-promedio">
                                <span id="promedioTexto">4.8</span> 
                                <span class="material-icons" style="font-size:19px; color:#ffd700;">star</span> 
                                <span id="totalVotosTexto" style="color:var(--texto-secundario); font-size:0.78rem;">(80 valoraciones)</span>
                            </div>

                            <div class="stars-container" id="starsContainer">
                                <span class="material-icons star-icon" data-value="1" onclick="calificar(1)">star</span>
                                <span class="material-icons star-icon" data-value="2" onclick="calificar(2)">star</span>
                                <span class="material-icons star-icon" data-value="3" onclick="calificar(3)">star</span>
                                <span class="material-icons star-icon" data-value="4" onclick="calificar(4)">star</span>
                                <span class="material-icons star-icon" data-value="5" onclick="calificar(5)">star</span>
                            </div>

                            <div id="votoMensaje" class="voto-mensaje"></div>
                        </div>

                        <div class="datos-grid" style="margin-bottom: 8px; text-align: center;">
                            <div class="dato-item full-width">
                                <span class="dato-val destacado">LINEA DIRECTA DE ATENCIÓN</span>
                                <span class="dato-val">+57 318 988 2787</span>

                            </div>
                        </div>

                        <p>Este carnet acredita la identidad y el <strong>Rol de Conductor Autorizado</strong> en la plataforma <strong>SegurApp Recorridos</strong> en Neiva, Huila.</p>
                    </div>

                    <div style="margin-top: 6px; text-align: center; font-size: 0.75rem; color: var(--texto-secundario);">
                        © 2026 SegurApp Recorridos. Todos los derechos reservados.
                    </div>
                </div>

            </div>
        </div>

        <!-- BOTONES DE ACCIÓN PRINCIPALES -->
        <div class="acciones-bar">
            <button class="btn-accion solicitar-wasap" id="btnSolicitarRecorrido" onclick="enviarUbicacionPorWhatsApp()">
                <span class="material-icons" style="font-size:26px;">location_on</span> Solicitar Recorrido Ya
            </button>

            <!-- BOTÓN DE ACCESO RÁPIDO AL PANEL DEL CONDUCTOR / CUENTAS -->
            <button class="btn-accion panel-conductor" onclick="abrirPanelConductor()">
                <span class="material-icons">calculate</span> Gestión de Cuentas (Conductor)
            </button>

            <div class="btn-grupo-principal">
                <button class="btn-accion guardar-contacto" onclick="guardarContactoVCF()">
                    <span class="material-icons">person_add</span> Contacto
                </button>
                <button class="btn-accion quienes-somos" onclick="abrirQuienesSomos()">
                    <span class="material-icons">info</span> Nosotros
                </button>
                <button class="btn-accion terminos-condiciones" onclick="abrirTerminos()">
                    <span class="material-icons">gavel</span> Términos
                </button>
            </div>

           
        </div>

        <!-- MODAL AUTENTICACIÓN (LOGIN / REGISTRO / RECUPERAR) -->
        <div class="modal-overlay" id="modalAuth">
            <div class="modal-contenido">
                <div class="modal-header">
                    <div class="modal-titulo">
                        <span class="material-icons">badge</span> Cuenta de Usuario
                    </div>
                    <button class="btn-cerrar" onclick="cerrarAuthModal()">
                        <span class="material-icons">close</span>
                    </button>
                </div>

                <div class="auth-tabs">
                    <button class="auth-tab-btn active" id="tabLogin" onclick="switchAuthTab('login')">Ingresar</button>
                    <button class="auth-tab-btn" id="tabRegister" onclick="switchAuthTab('register')">Registro</button>
                    <button class="auth-tab-btn" id="tabForgot" onclick="switchAuthTab('forgot')">Recuperar</button>
                </div>

                <!-- FORMULARIO LOGIN -->
                <form id="formLogin" onsubmit="procesarLogin(event)">
                    <div class="form-group">
                        <label>Número de Cédula (CC)</label>
                        <input type="number" id="loginCC" required placeholder="Ej: 1006506890">
                    </div>
                    <div class="form-group">
                        <label>Clave</label>
                        <input type="password" id="loginPass" required placeholder="********">
                    </div>
                    <span class="link-forgot-pass" onclick="switchAuthTab('forgot')">¿Olvidaste tu contraseña?</span>
                    <button type="submit" class="btn-submit-auth">Ingresar</button>
                </form>

                <!-- FORMULARIO REGISTRO -->
                <form id="formRegister" style="display: none;" onsubmit="procesarRegistro(event)">
                    <div class="avatar-upload-preview">
                        <img src="https://ui-avatars.com/api/?name=Foto&background=333&color=fff" id="previewFoto" class="preview-circle" alt="Preview">
                        <div style="flex: 1;">
                            <label style="font-size: 0.75rem; color: var(--dorado-brillante); font-weight: 700; text-transform: uppercase; display: block; margin-bottom: 4px;">Foto de Perfil</label>
                            <input type="file" id="regFoto" accept="image/*" onchange="convertirFotoBase64(this, 'previewFoto')" style="font-size: 0.75rem; color: #aaa; width: 100%;">
                        </div>
                    </div>

                    <div class="form-group">
                        <label>Nombre Completo *</label>
                        <input type="text" id="regNombre" required placeholder="Ej: Juan Pérez">
                    </div>
                    <div class="form-group">
                        <label>Cédula de Ciudadanía (CC) *</label>
                        <input type="number" id="regCC" required placeholder="Ej: 1006506890">
                    </div>
                    <div class="form-group">
                        <label>Número Celular *</label>
                        <input type="tel" id="regTel" required placeholder="Ej: 3189882787">
                    </div>
                    <div class="form-group">
                        <label>Contacto de Emergencia (Opcional)</label>
                        <input type="tel" id="regEmergencia" placeholder="Ej: 3150000000">
                    </div>
                    <div class="form-group">
                        <label>Clave *</label>
                        <input type="password" id="regPass" required placeholder="Crea una contraseña">
                    </div>
                    <button type="submit" class="btn-submit-auth">Crear Mi Cuenta</button>
                </form>

                <!-- FORMULARIO RECUPERAR CLAVE -->
                <form id="formForgot" style="display: none;" onsubmit="procesarRecuperacion(event)">
                    <p style="font-size:0.8rem; color:var(--texto-secundario); margin-bottom:12px;">
                        Ingresa tu número de Cédula y el Teléfono Celular asociado a tu cuenta para restaurar tu contraseña.
                    </p>
                    <div class="form-group">
                        <label>Cédula de Ciudadanía (CC) *</label>
                        <input type="number" id="forgotCC" required placeholder="Ej: 1006506890">
                    </div>
                    <div class="form-group">
                        <label>Número Celular Registrado *</label>
                        <input type="tel" id="forgotTel" required placeholder="Ej: 3189882787">
                    </div>
                    <div class="form-group">
                        <label>Nueva Contraseña *</label>
                        <input type="password" id="forgotNewPass" required placeholder="Nueva clave">
                    </div>
                    <button type="submit" class="btn-submit-auth">Restablecer Clave</button>
                </form>
            </div>
        </div>

        <!-- MODAL GESTIÓN DE CUENTAS / FINANZAS DEL CONDUCTOR -->
        <div class="modal-overlay" id="modalFinanzas">
            <div class="modal-contenido" style="max-width: 520px;">
                <div class="modal-header">
                    <div class="modal-titulo">
                        <span class="material-icons">account_balance_wallet</span> Cuentas del Conductor
                    </div>
                    <button class="btn-cerrar" onclick="cerrarPanelFinanzas()">
                        <span class="material-icons">close</span>
                    </button>
                </div>

                <div class="finanzas-dashboard">
                    <!-- Tarjetas resumen -->
                    <div class="resumen-cards-grid">
                        <div class="card-metrica">
                            <span class="titulo-metrica">Ingresos Totales</span>
                            <span class="valor-metrica ingreso" id="lblTotalIngresos">$0</span>
                        </div>
                        <div class="card-metrica">
                            <span class="titulo-metrica">Gastos (Gasolina/Manto)</span>
                            <span class="valor-metrica gasto" id="lblTotalGastos">$0</span>
                        </div>
                        <div class="card-metrica" style="grid-column: span 2;">
                            <span class="titulo-metrica">Balance Neto Operativo</span>
                            <span class="valor-metrica balance" id="lblBalanceNeto">$0</span>
                        </div>
                    </div>

                    <!-- SECCIÓN MAPA Y CÁLCULO DE RECORRIDO AUTOMÁTICO (MODERNO E INTERACTIVO) -->
                    <div class="seccion-mapa-recorrido">
                        <h4 style="font-size: 0.85rem; color: var(--dorado-brillante); text-transform: uppercase; display: flex; align-items: center; gap: 6px;">
                            <span class="material-icons" style="font-size: 18px;">map</span> Medidor Automático de Recorrido (Interactivo)
                        </h4>
                        
                        <!-- Barra de búsqueda / Guardar dirección -->
                        <div class="map-search-bar">
                            <input type="text" id="inputBuscarDireccion" placeholder="Buscar dirección (ej. Mi casa, Trabajo)...">
                            <button class="btn-map-action" onclick="buscarYMarcarDireccion()">
                                <span class="material-icons" style="font-size: 16px;">search</span> Buscar
                            </button>
                        </div>

                        <!-- Contenedor del Mapa Leaflet -->
                        <div id="mapContainer"></div>

                        <!-- Lista de Marcadores Guardados -->
                        <div style="font-size: 0.78rem; color: var(--dorado-brillante); font-weight: 700; text-transform: uppercase;">Marcadores Guardados (Casa / Trabajo / Clientes):</div>
                        <div class="marcadores-guardados-box" id="listaMarcadoresGuardados">
                            <div style="color: var(--texto-secundario); text-align: center; padding: 4px;">No hay marcadores guardados. Haz clic en el mapa para añadir uno.</div>
                        </div>

                        <div style="display: flex; justify-content: space-between; font-size: 0.82rem; color: #fff; background: #000; padding: 6px 10px; border-radius: 8px; border: 1px solid var(--borde-sutil);">
                            <span>Distancia: <strong id="lblDistanciaKm" style="color: var(--dorado-brillante);">0.00 km</strong></span>
                            <span>Costo Calculado: <strong id="lblCostoCalculado" style="color: var(--verde-verificado);">$0 COP</strong></span>
                        </div>
                        <div class="controles-recorrido">
                            <button class="btn-control-mapa" id="btnIniciarRecorrido" onclick="iniciarRastreoRecorrido()">
                                <span class="material-icons" style="font-size: 18px;">play_arrow</span> Iniciar Recorrido
                            </button>
                            <button class="btn-control-mapa detener" id="btnDetenerRecorrido" onclick="detenerRastreoRecorrido()" disabled style="opacity: 0.5;">
                                <span class="material-icons" style="font-size: 18px;">stop</span> Finalizar y Guardar
                            </button>
                        </div>
                    </div>

                    <!-- Formulario agregar transacción manual -->
                    <form id="formTransaccion" onsubmit="agregarTransaccion(event)" style="background: #0f1218; padding: 12px; border-radius: 12px; border: 1px solid var(--borde-sutil);">
                        <h4 style="font-size: 0.85rem; color: var(--dorado-brillante); margin-bottom: 8px; text-transform: uppercase;">Registrar Movimiento Manual</h4>
                        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 8px;">
                            <div class="form-group" style="margin-bottom: 6px;">
                                <label>Tipo</label>
                                <select id="transTipo" required>
                                    <option value="ingreso">Ingreso (Carrera)</option>
                                    <option value="gasto">Gasto (Gasolina/Otro)</option>
                                </select>
                            </div>
                            <div class="form-group" style="margin-bottom: 6px;">
                                <label>Monto ($)</label>
                                <input type="number" id="transMonto" required placeholder="Ej: 10000">
                            </div>
                        </div>
                        <div class="form-group" style="margin-bottom: 8px;">
                            <label>Descripción / Ruta</label>
                            <input type="text" id="transDesc" required placeholder="Ej: Carrera Centro - Sur">
                        </div>
                        <button type="submit" class="btn-submit-auth" style="margin-top: 0; padding: 8px;">Añadir Registro</button>
                    </form>

                    <!-- Historial -->
                    <div>
                        <h4 style="font-size: 0.85rem; color: #fff; margin-bottom: 6px; text-transform: uppercase;">Historial de Movimientos</h4>
                        <div class="seccion-transacciones" id="listaTransacciones">
                            <!-- Se llena dinámicamente -->
                        </div>
                    </div>

                 
                </div>
            </div>
        </div>

        <!-- MODAL EDITAR PERFIL DE USUARIO -->
        <div class="modal-overlay" id="modalEditUser">
            <div class="modal-contenido">
                <div class="modal-header">
                    <div class="modal-titulo">
                        <span class="material-icons">manage_accounts</span> Editar Perfil
                    </div>
                    <button class="btn-cerrar" onclick="cerrarEditModal()">
                        <span class="material-icons">close</span>
                    </button>
                </div>

                <form id="formEditUser" onsubmit="procesarEdicionUsuario(event)">
                    <div class="avatar-upload-preview">
                        <img src="https://ui-avatars.com/api/?name=Foto&background=333&color=fff" id="previewFotoEdit" class="preview-circle" alt="Preview Edit">
                        <div style="flex: 1;">
                            <label style="font-size: 0.75rem; color: var(--dorado-brillante); font-weight: 700; text-transform: uppercase; display: block; margin-bottom: 4px;">Cambiar Foto</label>
                            <input type="file" id="editFoto" accept="image/*" onchange="convertirFotoBase64(this, 'previewFotoEdit')" style="font-size: 0.75rem; color: #aaa; width: 100%;">
                        </div>
                    </div>

                    <div class="form-group">
                        <label>Nombre Completo</label>
                        <input type="text" id="editNombre" required>
                    </div>
                    <div class="form-group">
                        <label>Cédula (No editable)</label>
                        <input type="number" id="editCC" readonly style="opacity: 0.6; cursor: not-allowed;">
                    </div>
                    <div class="form-group">
                        <label>Número Celular</label>
                        <input type="tel" id="editTel" required>
                    </div>
                    <div class="form-group">
                        <label>Contacto de Emergencia</label>
                        <input type="tel" id="editEmergencia">
                    </div>
                    <div class="form-group">
                        <label>Nueva Clave (Opcional)</label>
                        <input type="password" id="editPass" placeholder="Dejar en blanco si no deseas cambiarla">
                    </div>
                    <button type="submit" class="btn-submit-auth">Guardar Cambios</button>
                </form>
            </div>
        </div>

        <!-- MODAL QUIÉNES SOMOS -->
        <div class="modal-overlay" id="modalQuienesSomos">
            <div class="modal-contenido">
                <div class="modal-header">
                    <div class="modal-titulo">
                        <span class="material-icons">groups</span> Quiénes Somos
                    </div>
                    <button class="btn-cerrar" onclick="cerrarQuienesSomos()">
                        <span class="material-icons">close</span>
                    </button>
                </div>
                <div class="modal-body">
                    <p>En <strong>SegurApp Recorridos</strong> nos dedicamos a transformar la movilización urbana en Neiva, ofreciendo un servicio de transporte individual express altamente seguro, rápido y confiable.</p>
                    
                    <h4><span class="material-icons">verified_user</span> Nuestra Misión</h4>
                    <p>Proporcionar un traslado rápido y seguro para cada pasajero, respaldado por conductores totalmente identificados y verificados en tiempo real.</p>

                    <h4><span class="material-icons">star</span> ¿Por qué elegirnos?</h4>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Seguridad Garantizada:</strong> Validación digital de la identidad del conductor e historial de vehículo.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Respuesta Inmediata:</strong> Ubicación precisa en tiempo real a través de WhatsApp para recogidas inmediatas.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Trato Personalizado:</strong> Calificación y servicio confiable en cada recorrido.
                    </div>
                </div>
            </div>
        </div>

        <!-- MODAL TÉRMINOS Y CONDICIONES (CON REGISTRO DE DATOS Y MANEJO DE INFORMACIÓN) -->
        <div class="modal-overlay" id="modalTerminos">
            <div class="modal-contenido">
                <div class="modal-header">
                    <div class="modal-titulo" style="display: flex; align-items: center; gap: 10px;">
                        <img src="img/png.png" alt="SegurApp Logo" style="height: 91px; width: auto; object-fit: contain;">
                        <span>Términos y Condiciones</span>
                    </div>
                    <button class="btn-cerrar" onclick="cerrarTerminos()">
                        <span class="material-icons">close</span>
                    </button>
                </div>
                <div class="modal-body">
                    <div class="alerta-seguridad-box">
                        <div class="alerta-header">
                            <div class="alerta-titulo-text">
                                <span class="material-icons" style="font-size: 20px;">shield</span> Protocolo Neiva
                            </div>
                            <div class="badge-live-seguridad">
                                <span class="dot-live"></span> Activo
                            </div>
                        </div>
                        <p class="alerta-body-desc">
                            Diseñado prioritariamente para proteger la integridad física y patrimonial del usuario y del conductor ante eventualidades de seguridad urbana.
                        </p>
                        <div class="alerta-puntos-claves">
                            <div class="punto-seguridad-item">
                                <span class="material-icons">my_location</span> Geolocalización activa requerida
                            </div>
                            <div class="punto-seguridad-item">
                                <span class="material-icons">badge</span> Verificación mutua de identidad
                            </div>
                            <div class="punto-seguridad-item">
                                <span class="material-icons">health_and_safety</span> Casco de protección obligatorio
                            </div>
                        </div>
                    </div>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">verified</span> 1. Identificación y Verificación Mutual</h4>
                    <p><strong>• Del Conductor:</strong> Se garantiza la identidad del conductor (Sergio Alejandro Tapiero Chala - C.C. 1.006.506.890) y del vehículo registrado (TVS Raider Negra - BWQ 69H).</p>
                    <p><strong>• Del Pasajero:</strong> El usuario debe proporcionar voluntariamente su ubicación GPS exacta en tiempo real antes de abordar. El conductor se reserva el derecho de verificar la identidad del pasajero (Cédula de Ciudadanía) antes de iniciar la marcha.</p>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">security</span> 2. Protocolo de Seguridad del Conductor</h4>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Cancelación por Inseguridad:</strong> El conductor puede rechazar o cancelar cualquier servicio si el punto de recogida o destino representa un riesgo inminente, zonas de alteración de orden público o barrios con restricciones operativas de seguridad.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Inspección Prevención:</strong> No se transportarán elementos sospechosos, maletas cerradas de procedencia dudosa, ni paquetes de gran volumen que comprometan la maniobrabilidad de la motocicleta o la seguridad vial.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Estado del Usuario:</strong> Se prohíbe rotundamente el traslado de personas bajo sospecha de porte de armas, sustancias psicoactivas o en estado de embriaguez extrema que pongan en riesgo la estabilidad del vehículo.
                    </div>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">health_and_safety</span> 3. Protocolo de Seguridad del Cliente (Pasajero)</h4>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Casco Obligatorio:</strong> Uso indispensable y correcto del casco de protección suministrado o propio durante todo el recorrido.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Rastreabilidad:</strong> Cada servicio genera un enlace directo de WhatsApp con coordenadas GPS que el pasajero puede compartir con familiares en tiempo real.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Monitoreo de Ruta:</strong> El recorrido seguirá estrictamente la ruta navegable más segura. No se realizarán desvíos no autorizados solicitados en la vía sin previa verificación de seguridad.
                    </div>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">dns</span> 4. Registro de Datos y Manejo de Información</h4>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Recolección y Autorización:</strong> Al registrarse, crear una cuenta o utilizar los servicios de <strong>SegurApp Recorridos</strong>, el usuario autoriza de manera expresa la recolección, almacenamiento y tratamiento de sus datos personales (nombre completo, número de cédula, teléfono de contacto, fotografía de perfil y datos de emergencia) conforme a las normativas vigentes de protección de datos.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Finalidad del Tratamiento:</strong> La información recopilada tiene como única finalidad garantizar la seguridad operativa de los trayectos, validar la identidad mutua entre pasajero y conductor, facilitar la comunicación inmediata vía canales digitales y llevar un registro transparente de las transacciones y servicios prestados en Neiva, Huila.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Confidencialidad y Seguridad de los Datos:</strong> <strong>SegurApp Recorridos</strong> implementa medidas de seguridad técnicas y administrativas razonables para proteger la información de accesos no autorizados. Los datos de geolocalización y registros financieros se manejan bajo estricta confidencialidad y no son comercializados con terceros.
                    </div>
                    <div class="caracteristica-box">
                        <strong style="color: #fff;">• Derechos del Titular:</strong> El usuario podrá en cualquier momento actualizar, rectificar o solicitar la supresión de sus datos personales registrados en la plataforma a través de los canales de atención autorizados de la aplicación.
                    </div>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">block</span> 5. Prohibiciones Generales</h4>
                    <p>• Prohibido sobrecupo (solo se permite 1 pasajero por viaje).</p>
                    <p>• Prohibido manipular dispositivos móviles o distraer al conductor durante el trayecto.</p>
                    <p>• Cero tolerancia a agresiones verbales o físicas por cualquiera de las partes.</p>

                    <h4 class="terminos-titulo-seccion"><span class="material-icons">gavel</span> 6. Jurisdicción y Aceptación</h4>
                    <p>Al hacer clic en <em>"Solicitar Recorrido Ya"</em>, registrarse o contactar vía WhatsApp, el usuario declara haber leído, comprendido y aceptado plenamente los presentes Términos, Condiciones y Políticas de Manejo de Información para la ciudad de Neiva, Huila.</p>

                    <button class="btn-aceptar-terminos" onclick="aceptarTerminosModal()">
                        <span class="material-icons">check_circle</span> Aceptar y Continuar
                    </button>
                </div>
            </div>
        </div>

        <!-- SCRIPT DE LÓGICA Y AUTENTICACIÓN -->
        <script>
            const STORAGE_KEY_VOTO = 'segurapp_user_voted';
            const STORAGE_KEY_START = 'segurapp_start_time';
            const STORAGE_KEY_TERMS = 'segurapp_terms_accepted_v1';
            const STORAGE_USERS = 'segurapp_registered_users';
            const STORAGE_CURRENT_USER = 'segurapp_current_logged_user';
            const STORAGE_FINANZAS = 'segurapp_driver_finances';
            const STORAGE_MARCADORES = 'segurapp_saved_markers_v1';
            
            const URL_APPS_SCRIPT = "https://script.google.com/macros/s/AKfycbxFfdvKCpwciO7odqxyqredyQWGf1MEvIh7CIZts8PoEySLxMxIxbqG51ST5_qEnvaF/exec";

            const BASE_VALORACIONES = 93;
            const HORAS_POR_INCREMENTO = 2.5;

            let fotoBase64Temp = '';

            /* --- SISTEMA DE ALERTAS Y LETREROS INTERACTIVOS (REEMPLAZO DE ALERT/PROMPT/CONFIRM NATIVOS) --- */
            function mostrarToast(titulo, mensaje, tipo = 'success') {
                const container = document.getElementById('toastContainer');
                const toast = document.createElement('div');
                toast.className = `custom-toast ${tipo}`;
                
                let icono = 'check_circle';
                if (tipo === 'error') icono = 'error';
                else if (tipo === 'info') icono = 'info';

                toast.innerHTML = `
                    <span class="material-icons toast-icon">${icono}</span>
                    <div class="toast-content">
                        <span class="toast-title">${titulo}</span>
                        <span class="toast-msg">${mensaje}</span>
                    </div>
                    <button class="toast-close" onclick="this.parentElement.remove()">&times;</button>
                `;

                container.appendChild(toast);
                setTimeout(() => { toast.classList.add('show'); }, 50);

                setTimeout(() => {
                    toast.classList.remove('show');
                    setTimeout(() => { toast.remove(); }, 400);
                }, 4000);
            }

            let resolveAlertaInteractiva = null;

            function mostrarAlertaInteractiva({ titulo, mensaje, tipo = 'info', esPrompt = false, esConfirm = false, valorInicial = '' }) {
                return new Promise((resolve) => {
                    resolveAlertaInteractiva = resolve;
                    const overlay = document.getElementById('interactiveAlertModal');
                    const titleEl = document.getElementById('interactiveAlertTitle');
                    const msgEl = document.getElementById('interactiveAlertMsg');
                    const iconEl = document.getElementById('interactiveAlertIcon');
                    const inputContainer = document.getElementById('interactiveAlertInputContainer');
                    const inputField = document.getElementById('interactiveAlertInputField');
                    const btnOk = document.getElementById('interactiveAlertBtnOk');
                    const btnCancel = document.getElementById('interactiveAlertBtnCancel');

                    titleEl.innerText = titulo;
                    msgEl.innerText = mensaje;

                    if (tipo === 'success') iconEl.innerText = 'check_circle';
                    else if (tipo === 'error') iconEl.innerText = 'error';
                    else iconEl.innerText = 'notifications_active';

                    if (esPrompt) {
                        inputContainer.style.display = 'block';
                        inputField.value = valorInicial;
                        btnCancel.style.display = 'block';
                        btnOk.innerText = 'Aceptar';
                        setTimeout(() => inputField.focus(), 100);
                    } else if (esConfirm) {
                        inputContainer.style.display = 'none';
                        btnCancel.style.display = 'block';
                        btnOk.innerText = 'Sí, Continuar';
                    } else {
                        inputContainer.style.display = 'none';
                        btnCancel.style.display = 'none';
                        btnOk.innerText = 'Aceptar';
                    }

                    overlay.classList.add('active');
                });
            }

            function cerrarAlertaInteractiva(resultado) {
                const overlay = document.getElementById('interactiveAlertModal');
                const inputField = document.getElementById('interactiveAlertInputField');
                overlay.classList.remove('active');

                if (resolveAlertaInteractiva) {
                    const inputContainer = document.getElementById('interactiveAlertInputContainer');
                    if (inputContainer.style.display === 'block') {
                        resolveAlertaInteractiva(resultado ? inputField.value : null);
                    } else {
                        resolveAlertaInteractiva(resultado);
                    }
                    resolveAlertaInteractiva = null;
                }
            }

            /* --- MANEJO DE SESIÓN Y REGISTRO --- */
            function obtenerUsuarios() {
                return JSON.parse(localStorage.getItem(STORAGE_USERS) || '[]');
            }

            function obtenerUsuarioActual() {
                return JSON.parse(localStorage.getItem(STORAGE_CURRENT_USER) || 'null');
            }

            function switchAuthTab(type) {
                const btnLogin = document.getElementById('tabLogin');
                const btnRegister = document.getElementById('tabRegister');
                const btnForgot = document.getElementById('tabForgot');
                
                const formLogin = document.getElementById('formLogin');
                const formRegister = document.getElementById('formRegister');
                const formForgot = document.getElementById('formForgot');

                btnLogin.classList.remove('active');
                btnRegister.classList.remove('active');
                btnForgot.classList.remove('active');

                formLogin.style.display = 'none';
                formRegister.style.display = 'none';
                formForgot.style.display = 'none';

                if (type === 'login') {
                    btnLogin.classList.add('active');
                    formLogin.style.display = 'block';
                } else if (type === 'register') {
                    btnRegister.classList.add('active');
                    formRegister.style.display = 'block';
                } else if (type === 'forgot') {
                    btnForgot.classList.add('active');
                    formForgot.style.display = 'block';
                }
            }

            function convertirFotoBase64(input, targetImgId) {
                if (input.files && input.files[0]) {
                    const reader = new FileReader();
                    reader.onload = function(e) {
                        fotoBase64Temp = e.target.result;
                        document.getElementById(targetImgId).src = fotoBase64Temp;
                    };
                    reader.readAsDataURL(input.files[0]);
                }
            }

            async function guardarEnGoogleSheets(datosUsuario, accion) {
                try {
                    await fetch(URL_APPS_SCRIPT, {
                        method: 'POST',
                        mode: 'no-cors',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ accion: accion, ...datosUsuario })
                    });
                } catch (error) {
                    console.error("Error al sincronizar con Google Sheets:", error);
                }
            }

            async function procesarRegistro(e) {
                e.preventDefault();
                const nombre = document.getElementById('regNombre').value.trim();
                const cc = document.getElementById('regCC').value.trim();
                const tel = document.getElementById('regTel').value.trim();
                const emergencia = document.getElementById('regEmergencia').value.trim();
                const pass = document.getElementById('regPass').value.trim();

                const usuarios = obtenerUsuarios();
                if (usuarios.some(u => u.cc === cc)) {
                    mostrarToast('Error de Registro', 'Ya existe un usuario registrado con esta cédula (CC).', 'error');
                    return;
                }

                const fotoFinal = fotoBase64Temp || `https://ui-avatars.com/api/?name=${encodeURIComponent(nombre)}&background=ffd700&color=000`;

                const nuevoUsuario = { nombre, cc, tel, emergencia, pass, foto: fotoFinal };

                usuarios.push(nuevoUsuario);
                localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
                localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(nuevoUsuario));

                await guardarEnGoogleSheets(nuevoUsuario, 'registro');

                fotoBase64Temp = '';
                mostrarToast('¡Éxito!', '¡Cuenta creada y guardada exitosamente!', 'success');
                cerrarAuthModal();
                cargarEstadoUsuario();
            }

            function procesarLogin(e) {
                e.preventDefault();
                const cc = document.getElementById('loginCC').value.trim();
                const pass = document.getElementById('loginPass').value.trim();

                if (cc === "1006506890" && (pass === "123456" || pass === "admin" || pass === "Sergio2026")) {
                    const driverUser = {
                        nombre: "Sergio Alejandro Tapiero Chala",
                        cc: "1006506890",
                        tel: "3189882787",
                        emergencia: "3150000000",
                        pass: pass,
                        foto: "https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/699116233_989437980648237_9201268186456313724_n.jpg"
                    };
                    localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(driverUser));
                    mostrarToast('Bienvenido', '¡Bienvenido, Conductor Oficial Sergio Tapiero!', 'success');
                    cerrarAuthModal();
                    cargarEstadoUsuario();
                    return;
                }

                const usuarios = obtenerUsuarios();
                const user = usuarios.find(u => u.cc === cc && u.pass === pass);

                if (user) {
                    localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(user));
                    mostrarToast('Bienvenido', `¡Bienvenido de nuevo, ${user.nombre}!`, 'success');
                    cerrarAuthModal();
                    cargarEstadoUsuario();
                } else {
                    mostrarToast('Acceso Denegado', 'Cédula o contraseña incorrectas.', 'error');
                }
            }

            async function procesarRecuperacion(e) {
                e.preventDefault();
                const cc = document.getElementById('forgotCC').value.trim();
                const tel = document.getElementById('forgotTel').value.trim();
                const newPass = document.getElementById('forgotNewPass').value.trim();

                let usuarios = obtenerUsuarios();
                const index = usuarios.findIndex(u => u.cc === cc && u.tel === tel);

                if (index !== -1) {
                    usuarios[index].pass = newPass;
                    localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
                    
                    const currentUser = obtenerUsuarioActual();
                    if (currentUser && currentUser.cc === cc) {
                        currentUser.pass = newPass;
                        localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(currentUser));
                    }

                    await guardarEnGoogleSheets(usuarios[index], 'actualizar');

                    mostrarToast('Restauración Exitosa', '¡Tu contraseña ha sido restablecida exitosamente!', 'success');
                    switchAuthTab('login');
                    document.getElementById('formForgot').reset();
                } else {
                    mostrarToast('Error', 'No se encontró ningún usuario que coincida con esa cédula y teléfono.', 'error');
                }
            }

            function abrirEditModal() {
                const user = obtenerUsuarioActual();
                if (!user) return;

                document.getElementById('editNombre').value = user.nombre;
                document.getElementById('editCC').value = user.cc;
                document.getElementById('editTel').value = user.tel;
                document.getElementById('editEmergencia').value = user.emergencia || '';
                document.getElementById('previewFotoEdit').src = user.foto;
                document.getElementById('editPass').value = '';
                fotoBase64Temp = '';

                document.getElementById('modalEditUser').classList.add('active');
            }

            function cerrarEditModal() {
                document.getElementById('modalEditUser').classList.remove('active');
            }

            async function procesarEdicionUsuario(e) {
                e.preventDefault();
                const currentUser = obtenerUsuarioActual();
                if (!currentUser) return;

                const nombre = document.getElementById('editNombre').value.trim();
                const tel = document.getElementById('editTel').value.trim();
                const emergencia = document.getElementById('editEmergencia').value.trim();
                const pass = document.getElementById('editPass').value.trim();

                let usuarios = obtenerUsuarios();
                const index = usuarios.findIndex(u => u.cc === currentUser.cc);

                if (index !== -1) {
                    usuarios[index].nombre = nombre;
                    usuarios[index].tel = tel;
                    usuarios[index].emergencia = emergencia;
                    if (fotoBase64Temp) usuarios[index].foto = fotoBase64Temp;
                    if (pass) usuarios[index].pass = pass;

                    localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
                    localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(usuarios[index]));
                    await guardarEnGoogleSheets(usuarios[index], 'actualizar');
                } else {
                    currentUser.nombre = nombre;
                    currentUser.tel = tel;
                    currentUser.emergencia = emergencia;
                    if (fotoBase64Temp) currentUser.foto = fotoBase64Temp;
                    if (pass) currentUser.pass = pass;
                    localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(currentUser));
                }

                fotoBase64Temp = '';
                mostrarToast('Perfil Actualizado', '¡Perfil actualizado con éxito!', 'success');
                cerrarEditModal();
                cargarEstadoUsuario();
            }

            function cargarEstadoUsuario() {
                const user = obtenerUsuarioActual();
                const barUserName = document.getElementById('barUserName');
                const barUserAvatar = document.getElementById('barUserAvatar');
                const lblBtnAuth = document.getElementById('lblBtnAuth');
                const btnEditUser = document.getElementById('btnEditUser');

                if (user) {
                    barUserName.innerText = user.nombre.split(' ')[0];
                    barUserAvatar.src = user.foto;
                    lblBtnAuth.innerText = 'Salir';
                    btnEditUser.style.display = 'flex';
                } else {
                    barUserName.innerText = 'Invitado';
                    barUserAvatar.src = 'https://ui-avatars.com/api/?name=Usuario&background=333&color=fff';
                    lblBtnAuth.innerText = 'Ingresar';
                    btnEditUser.style.display = 'none';
                }
            }

            async function abrirAuthModal() {
                const user = obtenerUsuarioActual();
                if (user) {
                    const confirmarCierre = await mostrarAlertaInteractiva({
                        titulo: 'Cerrar Sesión',
                        mensaje: `¿Deseas cerrar la sesión de ${user.nombre}?`,
                        tipo: 'info',
                        esConfirm: true
                    });
                    if (confirmarCierre) {
                        localStorage.removeItem(STORAGE_CURRENT_USER);
                        cargarEstadoUsuario();
                        mostrarToast('Sesión Cerrada', 'Has cerrado sesión correctamente.', 'success');
                    }
                } else {
                    switchAuthTab('login');
                    document.getElementById('modalAuth').classList.add('active');
                }
            }

            function cerrarAuthModal() {
                document.getElementById('modalAuth').classList.remove('active');
            }

            /* --- MÓDULO DE GESTIÓN DE CUENTAS / FINANZAS Y MAPA INTERACTIVO MODERNO --- */
            let mapaConductor = null;
            let marcadorConductor = null;
            let polylineRuta = null;
            let watchIdGPS = null;
            let distanciaTotalKm = 0;
            let ultimaPosicionGPS = null;
            let coordenadasRecorrido = [];
            let layerGroupMarcadores = null;

            const TARIFA_POR_KM_COP = 1389;

            async function abrirPanelConductor() {
                const user = obtenerUsuarioActual();
                if (!user || user.cc !== "6890") {
                    const passwordAdmin = await mostrarAlertaInteractiva({
                        titulo: 'Acceso Restringido',
                        mensaje: 'Ingresa la clave de conductor o cédula autorizada:',
                        tipo: 'info',
                        esPrompt: true
                    });
                    if (passwordAdmin !== "6890" && passwordAdmin !== "0408" && passwordAdmin !== "admin") {
                        mostrarToast('Acceso Denegado', 'Debes iniciar sesión como el conductor oficial.', 'error');
                        abrirAuthModal();
                        return;
                    }
                }
                actualizarDashboardFinanciero();
                document.getElementById('modalFinanzas').classList.add('active');
                
                setTimeout(() => {
                    inicializarMapaConductor();
                }, 300);
            }

            function cerrarPanelFinanzas() {
                detenerRastreoRecorrido();
                document.getElementById('modalFinanzas').classList.remove('active');
            }

            function inicializarMapaConductor() {
                const neivaCoords = [2.9273, -75.2819];
                if (!mapaConductor) {
                    mapaConductor = L.map('mapContainer').setView(neivaCoords, 14);
                    
                    L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
                        maxZoom: 19,
                        attribution: '&copy; <a href="https://carto.com/">CARTO</a>'
                    }).addTo(mapaConductor);

                    layerGroupMarcadores = L.layerGroup().addTo(mapaConductor);

                    marcadorConductor = L.marker(neivaCoords).addTo(mapaConductor)
                        .bindPopup("Mi ubicación en tiempo real (Conductor)").openPopup();
                    
                    polylineRuta = L.polyline([], {color: '#00ff88', weight: 4}).addTo(mapaConductor);

                    mapaConductor.on('click', async function(e) {
                        const nombreLugar = await mostrarAlertaInteractiva({
                            titulo: 'Nuevo Marcador',
                            mensaje: 'Nombre para este marcador (ej. Dirección casa de cliente X, Trabajo):',
                            tipo: 'info',
                            esPrompt: true
                        });
                        if (nombreLugar && nombreLugar.trim() !== "") {
                            guardarMarcadorPersonalizado(nombreLugar.trim(), e.latlng.lat, e.latlng.lng);
                        }
                    });
                } else {
                    mapaConductor.invalidateSize();
                }

                renderizarMarcadoresGuardados();
                actualizarUbicacionEnTiempoRealMapa();
            }

            function obtenerMarcadoresGuardados() {
                return JSON.parse(localStorage.getItem(STORAGE_MARCADORES) || '[]');
            }

            function guardarMarcadorPersonalizado(nombre, lat, lng) {
                const marcadores = obtenerMarcadoresGuardados();
                marcadores.push({ id: Date.now(), nombre, lat, lng });
                localStorage.setItem(STORAGE_MARCADORES, JSON.stringify(marcadores));
                renderizarMarcadoresGuardados();
                mostrarToast('Marcador Guardado', `¡Marcador "${nombre}" guardado con éxito!`, 'success');
            }

            async function eliminarMarcador(id) {
                const confirmar = await mostrarAlertaInteractiva({
                    titulo: 'Eliminar Marcador',
                    mensaje: '¿Estás seguro de eliminar este marcador guardado?',
                    tipo: 'error',
                    esConfirm: true
                });
                if (confirmar) {
                    let marcadores = obtenerMarcadoresGuardados();
                    marcadores = marcadores.filter(m => m.id !== id);
                    localStorage.setItem(STORAGE_MARCADORES, JSON.stringify(marcadores));
                    renderizarMarcadoresGuardados();
                    mostrarToast('Eliminado', 'Marcador eliminado correctamente.', 'success');
                }
            }

            function renderizarMarcadoresGuardados() {
                const marcadores = obtenerMarcadoresGuardados();
                const contenedor = document.getElementById('listaMarcadoresGuardados');
                contenedor.innerHTML = '';

                if (layerGroupMarcadores) {
                    layerGroupMarcadores.clearLayers();
                }

                if (marcadores.length === 0) {
                    contenedor.innerHTML = `<div style="color: var(--texto-secundario); text-align: center; padding: 4px;">No hay marcadores guardados. Haz clic en el mapa para añadir uno.</div>`;
                    return;
                }

                marcadores.forEach(m => {
                    if (layerGroupMarcadores) {
                        const marker = L.marker([m.lat, m.lng]).bindPopup(`<b>${m.nombre}</b><br>Lat: ${m.lat.toFixed(4)}, Lng: ${m.lng.toFixed(4)}`);
                        layerGroupMarcadores.addLayer(marker);
                    }

                    const pill = document.createElement('div');
                    pill.className = 'marcador-pill';
                    pill.innerHTML = `
                        <span class="marcador-nombre" onclick="centrarMapaEn(${m.lat}, ${m.lng})">📍 ${m.nombre}</span>
                        <span class="material-icons" style="font-size: 16px; color: #ff6666; cursor: pointer;" onclick="eliminarMarcador(${m.id})" title="Eliminar">delete</span>
                    `;
                    contenedor.appendChild(pill);
                });
            }

            function centrarMapaEn(lat, lng) {
                if (mapaConductor) {
                    mapaConductor.setView([lat, lng], 16);
                }
            }

            async function buscarYMarcarDireccion() {
                const query = document.getElementById('inputBuscarDireccion').value.trim();
                if (!query) return;

                try {
                    const response = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query + ", Neiva, Huila")}`);
                    const data = await response.json();

                    if (data && data.length > 0) {
                        const lat = parseFloat(data[0].lat);
                        const lon = parseFloat(data[0].lon);
                        
                        mapaConductor.setView([lat, lon], 16);
                        const nombrePersonalizado = await mostrarAlertaInteractiva({
                            titulo: 'Dirección Encontrada',
                            mensaje: `Dirección: ${data[0].display_name}\n\nIngresa un nombre para guardarla (ej. Casa de Juan):`,
                            tipo: 'success',
                            esPrompt: true,
                            valorInicial: query
                        });
                        if (nombrePersonalizado) {
                            guardarMarcadorPersonalizado(nombrePersonalizado, lat, lon);
                        }
                    } else {
                        mostrarToast('No Encontrado', 'No se encontró la dirección en Neiva. Intenta con más detalles.', 'error');
                    }
                } catch (err) {
                    console.error("Error buscando dirección:", err);
                    mostrarToast('Error de Conexión', 'Error al conectar con el servicio de geocodificación.', 'error');
                }
            }

            function actualizarUbicacionEnTiempoRealMapa() {
                if ("geolocation" in navigator) {
                    navigator.geolocation.getCurrentPosition(
                        (position) => {
                            const lat = position.coords.latitude;
                            const lng = position.coords.longitude;
                            if (marcadorConductor && mapaConductor) {
                                marcadorConductor.setLatLng([lat, lng]);
                                mapaConductor.setView([lat, lng], 15);
                            }
                        },
                        (err) => { console.warn("No se pudo obtener ubicación en tiempo real para el mapa:", err); },
                        { enableHighAccuracy: true }
                    );
                }
            }

            function calcularDistanciaMetros(lat1, lon1, lat2, lon2) {
                const R = 6371e3;
                const dLat = (lat2 - lat1) * Math.PI / 180;
                const dLon = (lon2 - lon1) * Math.PI / 180;
                const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                        Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                        Math.sin(dLon/2) * Math.sin(dLon/2);
                const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
                return R * c;
            }

            function iniciarRastreoRecorrido() {
                if (!navigator.geolocation) {
                    mostrarToast('No Compatible', 'Tu navegador no soporta geolocalización.', 'error');
                    return;
                }

                distanciaTotalKm = 0;
                coordenadasRecorrido = [];
                ultimaPosicionGPS = null;
                document.getElementById('lblDistanciaKm').innerText = "0.00 km";
                document.getElementById('lblCostoCalculado').innerText = "$0 COP";

                document.getElementById('btnIniciarRecorrido').disabled = true;
                document.getElementById('btnIniciarRecorrido').style.opacity = "0.5";
                document.getElementById('btnDetenerRecorrido').disabled = false;
                document.getElementById('btnDetenerRecorrido').style.opacity = "1";

                watchIdGPS = navigator.geolocation.watchPosition(
                    (position) => {
                        const lat = position.coords.latitude;
                        const lng = position.coords.longitude;
                        const nuevaPos = [lat, lng];

                        coordenadasRecorrido.push(nuevaPos);
                        if (polylineRuta) {
                            polylineRuta.setLatLngs(coordenadasRecorrido);
                        }
                        if (marcadorConductor) {
                            marcadorConductor.setLatLng(nuevaPos);
                        }
                        if (mapaConductor) {
                            mapaConductor.setView(nuevaPos, 16);
                        }

                        if (ultimaPosicionGPS) {
                            const distanciaMetros = calcularDistanciaMetros(
                                ultimaPosicionGPS[0], ultimaPosicionGPS[1],
                                lat, lng
                            );
                            if (distanciaMetros > 3) {
                                distanciaTotalKm += distanciaMetros / 1000;
                            }
                        }
                        ultimaPosicionGPS = nuevaPos;

                        const distanciaStr = distanciaTotalKm.toFixed(2);
                        const costoEst = Math.round(distanciaTotalKm * TARIFA_POR_KM_COP);
                        document.getElementById('lblDistanciaKm').innerText = distanciaStr + " km";
                        document.getElementById('lblCostoCalculado').innerText = "$" + costoEst.toLocaleString() + " COP";
                    },
                    (err) => { console.error("Error en geolocalización de ruta:", err); },
                    { enableHighAccuracy: true, maximumAge: 0, timeout: 5000 }
                );

                mostrarToast('Recorrido Iniciado', '¡Rastreo de recorrido iniciado!', 'success');
            }

            function detenerRastreoRecorrido() {
                if (watchIdGPS !== null) {
                    navigator.geolocation.clearWatch(watchIdGPS);
                    watchIdGPS = null;
                }

                document.getElementById('btnIniciarRecorrido').disabled = false;
                document.getElementById('btnIniciarRecorrido').style.opacity = "1";
                document.getElementById('btnDetenerRecorrido').disabled = true;
                document.getElementById('btnDetenerRecorrido').style.opacity = "0.5";

                if (distanciaTotalKm > 0.05) {
                    const costoFinal = Math.round(distanciaTotalKm * TARIFA_POR_KM_COP);
                    const desc = `Recorrido Automático (${distanciaTotalKm.toFixed(2)} km)`;
                    
                    const transacciones = obtenerTransaccionesFinancieras();
                    transacciones.unshift({
                        id: Date.now(),
                        tipo: 'ingreso',
                        monto: costoFinal,
                        desc: desc,
                        fecha: new Date().toLocaleString()
                    });
                    localStorage.setItem(STORAGE_FINANZAS, JSON.stringify(transacciones));
                    actualizarDashboardFinanciero();
                    mostrarToast('Recorrido Guardado', `¡Recorrido finalizado!\nDistancia: ${distanciaTotalKm.toFixed(2)} km\nIngreso: $${costoFinal.toLocaleString()} COP`, 'success');
                } else {
                    mostrarToast('Aviso', 'La distancia recorrida fue muy corta para registrar automáticamente.', 'info');
                }
            }

            function obtenerTransaccionesFinancieras() {
                return JSON.parse(localStorage.getItem(STORAGE_FINANZAS) || '[]');
            }

            function agregarTransaccion(e) {
                e.preventDefault();
                const tipo = document.getElementById('transTipo').value;
                const monto = parseFloat(document.getElementById('transMonto').value);
                const desc = document.getElementById('transDesc').value.trim();

                if (!monto || isNaN(monto) || !desc) return;

                const transacciones = obtenerTransaccionesFinancieras();
                transacciones.unshift({
                    id: Date.now(),
                    tipo,
                    monto,
                    desc,
                    fecha: new Date().toLocaleString()
                });

                localStorage.setItem(STORAGE_FINANZAS, JSON.stringify(transacciones));
                document.getElementById('formTransaccion').reset();
                actualizarDashboardFinanciero();
                mostrarToast('Movimiento Registrado', 'Transacción añadida exitosamente.', 'success');
            }

            async function eliminarTransaccion(id) {
                const confirmar = await mostrarAlertaInteractiva({
                    titulo: 'Eliminar Movimiento',
                    mensaje: '¿Deseas eliminar este registro financiero?',
                    tipo: 'error',
                    esConfirm: true
                });
                if (confirmar) {
                    let transacciones = obtenerTransaccionesFinancieras();
                    transacciones = transacciones.filter(t => t.id !== id);
                    localStorage.setItem(STORAGE_FINANZAS, JSON.stringify(transacciones));
                    actualizarDashboardFinanciero();
                    mostrarToast('Eliminado', 'Transacción eliminada correctamente.', 'success');
                }
            }

            function actualizarDashboardFinanciero() {
                const transacciones = obtenerTransaccionesFinancieras();
                let totalIngresos = 0;
                let totalGastos = 0;

                const listaHTML = document.getElementById('listaTransacciones');
                listaHTML.innerHTML = '';

                if (transacciones.length === 0) {
                    listaHTML.innerHTML = `<div style="text-align: center; color: var(--texto-secundario); padding: 10px; font-size: 0.8rem;">No hay movimientos registrados.</div>`;
                }

                transacciones.forEach(t => {
                    if (t.tipo === 'ingreso') totalIngresos += t.monto;
                    else totalGastos += t.monto;

                    const item = document.createElement('div');
                    item.className = 'transaccion-item';
                    item.innerHTML = `
                        <div class="transaccion-info">
                            <span class="transaccion-desc">${t.desc}</span>
                            <span class="transaccion-fecha">${t.fecha}</span>
                        </div>
                        <div style="display: flex; align-items: center; gap: 8px;">
                            <span class="transaccion-monto ${t.tipo}">${t.tipo === 'ingreso' ? '+' : '-'}$${t.monto.toLocaleString()}</span>
                            <span class="material-icons" style="font-size: 16px; color: #ff6666; cursor: pointer;" onclick="eliminarTransaccion(${t.id})" title="Eliminar">delete</span>
                        </div>
                    `;
                    listaHTML.appendChild(item);
                });

                const balanceNeto = totalIngresos - totalGastos;

                document.getElementById('lblTotalIngresos').innerText = "$" + totalIngresos.toLocaleString() + " COP";
                document.getElementById('lblTotalGastos').innerText = "$" + totalGastos.toLocaleString() + " COP";
                
                const lblBalance = document.getElementById('lblBalanceNeto');
                lblBalance.innerText = "$" + balanceNeto.toLocaleString() + " COP";
                lblBalance.className = "valor-metrica balance " + (balanceNeto >= 0 ? "ingreso" : "gasto");
            }

            function exportarCuentasCSV() {
                const transacciones = obtenerTransaccionesFinancieras();
                if (transacciones.length === 0) {
                    mostrarToast('Aviso', 'No hay datos para exportar.', 'info');
                    return;
                }

                let csvContent = "data:text/csv;charset=utf-8,ID,TIPO,MONTO (COP),DESCRIPCION,FECHA\n";
                transacciones.forEach(t => {
                    csvContent += `${t.id},${t.tipo},${t.monto},"${t.desc}","${t.fecha}"\n`;
                });

                const encodedUri = encodeURI(csvContent);
                const link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                link.setAttribute("download", `Cuentas_SegurApp_${new Date().toISOString().slice(0,10)}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                mostrarToast('Exportado', 'Archivo CSV descargado con éxito.', 'success');
            }

            function exportarCuentasImagen() {
                const modalContent = document.querySelector('#modalFinanzas .modal-contenido');
                html2canvas(modalContent, { scale: 2, backgroundColor: '#000000' }).then(canvas => {
                    const link = document.createElement('a');
                    link.download = `Cuentas_SegurApp_${new Date().toISOString().slice(0,10)}.png`;
                    link.href = canvas.toDataURL('image/png');
                    link.click();
                    mostrarToast('Exportado', 'Imagen de cuentas descargada con éxito.', 'success');
                });
            }

            /* --- FUNCIONES DEL CARNET Y NAVEGACIÓN --- */
            function voltearCarnet() {
                const escena = document.getElementById('escenaCarnet');
                escena.classList.toggle('flipped');
            }

            function enviarUbicacionPorWhatsApp() {
                const numeroConductor = "573189882787";
                
                if ("geolocation" in navigator) {
                    navigator.geolocation.getCurrentPosition(
                        (position) => {
                            const lat = position.coords.latitude;
                            const lon = position.coords.longitude;
                            const mapaLink = `https://maps.google.com/?q=${lat},${lon}`;
                            const mensaje = `¡Hola! 👋 Solicito un recorrido urgente en *SegurApp Recorridos*. Mi ubicación GPS actual es: ${mapaLink}`;
                            window.open(`https://wa.me/${numeroConductor}?text=${encodeURIComponent(mensaje)}`, '_blank');
                        },
                        (error) => {
                            const mensaje = `¡Hola! 👋 Solicito un recorrido urgente en *SegurApp Recorridos*. (No se pudo compartir la ubicación GPS automáticamente).`;
                            window.open(`https://wa.me/${numeroConductor}?text=${encodeURIComponent(mensaje)}`, '_blank');
                        },
                        { enableHighAccuracy: true, timeout: 10000 }
                    );
                } else {
                    const mensaje = `¡Hola! 👋 Solicito un recorrido urgente en *SegurApp Recorridos*.`;
                    window.open(`https://wa.me/${numeroConductor}?text=${encodeURIComponent(mensaje)}`, '_blank');
                }
            }

            function guardarContactoVCF() {
                const vcard = `BEGIN:VCARD
    VERSION:3.0
    FN:Sergio Alejandro Tapiero Chala (SegurApp)
    N:Chala;Sergio;Alejandro;Tapiero;
    TEL;TYPE=CELL:+573189882787
    ORG:SegurApp Recorridos
    TITLE:Gerente de Operaciones / Conductor Oficial
    NOTE:Transporte seguro y confiable en Neiva, Huila. TVS Raider Negra BWQ 69H.
    END:VCARD`;

                const blob = new Blob([vcard], { type: 'text/vcard;charset=utf-8' });
                const url = window.URL.createObjectURL(blob);
                const link = document.createElement('a');
                link.href = url;
                link.setAttribute('download', 'Sergio_Tapiero_SegurApp.vcf');
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                mostrarToast('Contacto Guardado', 'Archivo de contacto VCF descargado con éxito.', 'success');
            }

            function descargarCarnet() {
                const carnetElement = document.getElementById('escenaCarnet');
                html2canvas(carnetElement, { scale: 2, useCORS: true, backgroundColor: null }).then(canvas => {
                    const link = document.createElement('a');
                    link.download = 'Carnet_Digital_SegurApp.png';
                    link.href = canvas.toDataURL('image/png');
                    link.click();
                    mostrarToast('Descarga Exitosa', 'El carnet digital ha sido descargado como imagen.', 'success');
                });
            }

            function abrirQuienesSomos() {
                document.getElementById('modalQuienesSomos').classList.add('active');
            }

            function cerrarQuienesSomos() {
                document.getElementById('modalQuienesSomos').classList.remove('active');
            }

            function abrirTerminos() {
                document.getElementById('modalTerminos').classList.add('active');
            }

            function cerrarTerminos() {
                document.getElementById('modalTerminos').classList.remove('active');
            }

            function aceptarTerminosModal() {
                localStorage.setItem(STORAGE_KEY_TERMS, 'true');
                cerrarTerminos();
                mostrarToast('Aceptado', '¡Términos y Condiciones aceptados con éxito!', 'success');
            }

            function calcularValoracionesDinamicas() {
                const ahora = new Date().getTime();
                let inicio = localStorage.getItem(STORAGE_KEY_START);
                if (!inicio) {
                    inicio = ahora;
                    localStorage.setItem(STORAGE_KEY_START, inicio);
                }
                const horasTranscurridas = (ahora - parseInt(inicio)) / (1000 * 60 * 60);
                const incremento = Math.floor(horasTranscurridas / HORAS_POR_INCREMENTO);
                return BASE_VALORACIONES + incremento;
            }

            function actualizarEstrellasUI(votoUsuario) {
                const stars = document.querySelectorAll('.star-icon');
                stars.forEach(star => {
                    const val = parseInt(star.getAttribute('data-value'));
                    if (val <= votoUsuario) {
                        star.classList.add('active');
                    } else {
                        star.classList.remove('active');
                    }
                });
            }

            function calificar(voto) {
                localStorage.setItem(STORAGE_KEY_VOTO, voto);
                actualizarEstrellasUI(voto);
                
                const totalVotos = calcularValoracionesDinamicas();
                document.getElementById('totalVotosTexto').innerText = `(${totalVotos} valoraciones)`;
                document.getElementById('votoMensaje').innerText = `¡Gracias por calificar con ${voto} estrellas! ⭐`;
                mostrarToast('Calificación Registrada', `¡Has calificado con ${voto} estrellas!`, 'success');
                
                setTimeout(() => {
                    document.getElementById('votoMensaje').innerText = '';
                }, 4000);
            }

            window.onload = function() {
                cargarEstadoUsuario();
                
                const totalVotos = calcularValoracionesDinamicas();
                document.getElementById('totalVotosTexto').innerText = `(${totalVotos} valoraciones)`;

                const votoGuardado = localStorage.getItem(STORAGE_KEY_VOTO);
                if (votoGuardado) {
                    actualizarEstrellasUI(parseInt(votoGuardado));
                }
            };
        </script>
    </body> 
    </html>
