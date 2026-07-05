# ============================================================
# BYTEC US PRO - SPECIFICACION TECNICA COMPLETA
# Version: 2.0.0
# Empresa: ByteC Medical Devices
# Fecha: 2025-07-05
# ============================================================

# TABLA DE CONTENIDOS

1. [RESUMEN EJECUTIVO](#1-resumen-ejecutivo)
2. [ARQUITECTURA DEL SISTEMA](#2-arquitectura-del-sistema)
3. [ESTRUCTURA DE ARCHIVOS](#3-estructura-de-archivos)
4. [MODULO: SPLASH SCREEN](#4-modulo-splash-screen)
5. [MODULO: REBRAND COMPLETO](#5-modulo-rebrand)
6. [MODULO: UI MODERNA](#6-modulo-ui)
7. [MODULO: CONEXION PROBE](#7-modulo-conexion)
8. [MODULO: IMAGING](#8-modulo-imaging)
9. [MODULO: DICOM](#9-modulo-dicom)
10. [MODULO: PATIENT DB](#10-modulo-patient-db)
11. [MODULO: MEDIDAS](#11-modulo-medidas)
12. [MODULO: REPORTES](#12-modulo-reportes)
13. [MODULO: SETTINGS](#13-modulo-settings)
14. [MODULO: SEGURIDAD](#14-modulo-seguridad)
15. [BUILD Y DEPLOY](#15-build-deploy)
16. [TESTING](#16-testing)
17. [REGULACIONES](#17-regulaciones)
18. [APPENDICES](#18-appendices)

---

# 1. RESUMEN EJECUTIVO

## 1.1 Que es ByteC US Pro

ByteC US Pro es una aplicacion Android de ultrasonido medico inalambrico que se conecta a probes via WiFi/USB y ofrece imaging en tiempo real con soporte DICOM para integracion con PACS hospitalarios.

## 1.2 Diferencias con el original

| Aspecto | DolphinUSG (Original) | ByteC US Pro (Modificado) |
|---------|----------------------|---------------------------|
| Package | com.sonoptek.wirelessusg3 | com.bytec.uspro |
| Nombre | DolphinUSG | ByteC US Pro |
| Splash | No tiene | Animacion SVG 3 seg |
| Colores | Generic medical | ByteC Brand (Azul #1E88E5) |
| Icono | Sonostar generic | ByteC logo custom |
| DB local | No tiene | SQLite para pacientes |
| Multi-probe | Limitado | Expandido |
| Cloud backup | No tiene | Opcional |
| Reports | Basico | PDF profesional |
| Seguridad | Basica | Encriptacion + TLS |
| Idioma | Chingles | ESP/EN/PT |

## 1.3 Stack Tecnico

| Componente | Tecnologia |
|------------|-----------|
| Platform | Android 6.0+ (API 23) |
| Language | Java (smali) |
| Build | apktool + jarsigner |
| DICOM | dcm4che3 |
| Image | RenderScript |
| DB | SQLite |
| UI | Android XML |
| Anim | Lottie / SVG |

## 1.4 Requisitos Previos

```
- Java JDK 8+
- Android SDK (build-tools 30+)
- apktool 2.7+
- jarsigner (JDK)
- zipalign (Android SDK)
- keytool (JDK)
- ImageMagick (para SVG→PNG)
- Android Debug Bridge (adb)
```

---

# 2. ARQUITECTURA DEL SISTEMA

## 2.1 Diagrama de Componentes

```
┌─────────────────────────────────────────────────────────────────┐
│                        BYTEC US PRO                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   SPLASH    │  │   MAIN      │  │   SETTINGS  │            │
│  │   ACTIVITY  │──▶  ACTIVITY   │──▶  ACTIVITY   │            │
│  └─────────────┘  └──────┬──────┘  └─────────────┘            │
│                          │                                      │
│  ┌───────────────────────┼───────────────────────┐             │
│  │                       │                       │             │
│  ▼                       ▼                       ▼             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  PATIENT    │  │  IMAGING    │  │  DICOM      │            │
│  │  ACTIVITY   │  │  ENGINE     │  │  SERVICE    │            │
│  └─────────────┘  └──────┬──────┘  └──────┬──────┘            │
│                          │                │                     │
│  ┌───────────────────────┼────────────────┼──────────┐         │
│  │                       │                │          │         │
│  ▼                       ▼                ▼          ▼         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  PROBE      │  │  IMAGE      │  │  PACS       │            │
│  │  HANDLER    │  │  PROCESSOR  │  │  SERVER     │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 2.2 Flujo de Datos

```
┌──────────────────────────────────────────────────────────────┐
│                     FLOW DE DATOS                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PROBE ──WiFi/USB──▶ FRAME BUFFER ──▶ DSC ──▶ ENHANCE      │
│                                                │             │
│                                                ▼             │
│                                         ┌─────────────┐      │
│                                         │   DISPLAY   │      │
│                                         └─────────────┘      │
│                                                │             │
│                                    ┌───────────┼───────────┐ │
│                                    ▼           ▼           ▼ │
│                              ┌─────────┐ ┌─────────┐ ┌─────────┐
│                              │ CAPTURE │ │ DICOM   │ │ REPORT  │
│                              └─────────┘ └─────────┘ └─────────┘
│                                    │           │           │
│                                    ▼           ▼           ▼ │
│                              ┌─────────┐ ┌─────────┐ ┌─────────┐
│                              │ GALLERY │ │  PACS   │ │   PDF   │
│                              └─────────┘ └─────────┘ └─────────┘
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 2.3 Capas de la Aplicacion

| Capa | Responsabilidad | Archivos |
|------|----------------|----------|
| Presentation | UI, Activities, Views | *.java (layouts) |
| Business | Logica de negocio | *Service.java |
| Data | Acceso a datos, network | *Handler.java |
| Native | USB, WiFi, RenderScript | *.so, JNI |

---

# 3. ESTRUCTURA DE ARCHIVOS

## 3.1 Directorio Completo

```
bytec_us_pro/
│
├── original/                           # APK fuente
│   └── dolphin/                        # Decompilado actual
│       ├── AndroidManifest.xml
│       ├── classes.dex
│       ├── lib/
│       ├── assets/
│       └── res/
│
├── modified/                           # APK modificado
│   │
│   ├── AndroidManifest.xml             # MODIFICADO
│   ├── apktool.yml                     # Config apktool
│   │
│   ├── smali/                          # Codigo smali
│   │   └── com/
│   │       └── bytec/
│   │           └── uspro/
│   │               ├── MainActivity.smali
│   │               ├── SplashActivity.smali
│   │               ├── PatientActivity.smali
│   │               ├── ReportActivity.smali
│   │               ├── SettingActivity.smali
│   │               ├── MyApplication.smali
│   │               ├── ProbeManager.smali
│   │               ├── PatientDatabase.smali
│   │               ├── DicomService.smali
│   │               ├── ImageProcessor.smali
│   │               ├── AnnotationManager.smali
│   │               ├── CloudService.smali
│   │               ├── SettingsManager.smali
│   │               └── Utils.smali
│   │
│   ├── smali_classes2/                 # Clases adicionales
│   │   └── org/
│   │       └── dcm4che3/              # DICOM library
│   │
│   ├── res/                            # Recursos
│   │   ├── anim/
│   │   │   ├── fade_in.xml
│   │   │   ├── fade_out.xml
│   │   │   ├── slide_up.xml
│   │   │   ├── slide_down.xml
│   │   │   ├── scale_in.xml
│   │   │   └── bounce.xml
│   │   │
│   │   ├── animator/
│   │   │   ├── splash_scale.json      # Lottie
│   │   │   ├── probe_connect.json     # Lottie
│   │   │   └── loading.json           # Lottie
│   │   │
│   │   ├── color/
│   │   │   ├── bytec_colors.xml
│   │   │   ├── status_colors.xml
│   │   │   ├── doppler_colors.xml
│   │   │   └── ui_colors.xml
│   │   │
│   │   ├── drawable/
│   │   │   ├── bytec_logo.xml         # Vector logo
│   │   │   ├── bytec_logo_circle.xml  # Logo circular
│   │   │   ├── ic_mode_b.xml          # Icono B-Mode
│   │   │   ├── ic_mode_m.xml          # Icono M-Mode
│   │   │   ├── ic_mode_color.xml      # Icono Color
│   │   │   ├── ic_mode_pw.xml         # Icono PW
│   │   │   ├── ic_freeze.xml          # Icono freeze
│   │   │   ├── ic_unfreeze.xml        # Icono unfreeze
│   │   │   ├── ic_capture.xml         # Icono captura
│   │   │   ├── ic_measure.xml         # Icono medida
│   │   │   ├── ic_export.xml          # Icono export
│   │   │   ├── ic_report.xml          # Icono reporte
│   │   │   ├── ic_settings.xml        # Icono settings
│   │   │   ├── ic_patient.xml         # Icono paciente
│   │   │   ├── ic_dicom.xml           # Icono DICOM
│   │   │   ├── ic_probe.xml           # Icono probe
│   │   │   ├── ic_wifi.xml            # Icono wifi
│   │   │   ├── ic_usb.xml             # Icono usb
│   │   │   ├── ic_cloud.xml           # Icono cloud
│   │   │   ├── ic_save.xml            # Icono guardar
│   │   │   ├── ic_delete.xml          # Icono borrar
│   │   │   ├── ic_search.xml          # Icono buscar
│   │   │   ├── ic_add.xml             # Icono agregar
│   │   │   ├── ic_edit.xml            # Icono editar
│   │   │   ├── ic_share.xml           # Icono compartir
│   │   │   ├── ic_menu.xml            # Icono menu
│   │   │   ├── ic_back.xml            # Icono atras
│   │   │   ├── ic_close.xml           # Icono cerrar
│   │   │   ├── ic_check.xml           # Icono check
│   │   │   ├── ic_warning.xml         # Icono warning
│   │   │   ├── ic_error.xml           # Icono error
│   │   │   ├── ic_success.xml         # Icono success
│   │   │   ├── bg_button_primary.xml  # Boton primario
│   │   │   ├── bg_button_secondary.xml# Boton secundario
│   │   │   ├── bg_card.xml            # Card background
│   │   │   ├── bg_input.xml           # Input background
│   │   │   ├── bg_splash.xml          # Splash background
│   │   │   ├── bg_gradient.xml        # Gradiente
│   │   │   ├── divider.xml            # Linea divisora
│   │   │   └── ripple.xml             # Ripple effect
│   │   │
│   │   ├── drawable-hdpi/
│   │   │   ├── ic_launcher.png        # 72x72
│   │   │   ├── ic_launcher_round.png  # 72x72 round
│   │   │   └── bytec_logo.png         # Logo HD
│   │   │
│   │   ├── drawable-xhdpi/
│   │   │   ├── ic_launcher.png        # 96x96
│   │   │   ├── ic_launcher_round.png  # 96x96 round
│   │   │   └── bytec_logo.png         # Logo XHD
│   │   │
│   │   ├── drawable-xxhdpi/
│   │   │   ├── ic_launcher.png        # 144x144
│   │   │   ├── ic_launcher_round.png  # 144x144 round
│   │   │   └── bytec_logo.png         # Logo XXHD
│   │   │
│   │   ├── drawable-xxxhdpi/
│   │   │   ├── ic_launcher.png        # 192x192
│   │   │   ├── ic_launcher_round.png  # 192x192 round
│   │   │   └── bytec_logo.png         # Logo XXXHD
│   │   │
│   │   ├── layout/
│   │   │   ├── activity_splash.xml
│   │   │   ├── activity_main.xml
│   │   │   ├── activity_patient.xml
│   │   │   ├── activity_report.xml
│   │   │   ├── activity_settings.xml
│   │   │   ├── activity_dicom.xml
│   │   │   ├── activity_worklist.xml
│   │   │   ├── activity_gallery.xml
│   │   │   ├── dialog_patient_form.xml
│   │   │   ├── dialog_probe_select.xml
│   │   │   ├── dialog_dicom_config.xml
│   │   │   ├── dialog_measurement.xml
│   │   │   ├── dialog_export.xml
│   │   │   ├── dialog_confirm.xml
│   │   │   ├── item_patient.xml
│   │   │   ├── item_study.xml
│   │   │   ├── item_image.xml
│   │   │   ├── item_worklist.xml
│   │   │   ├── view_ultrasound.xml
│   │   │   ├── view_controls.xml
│   │   │   ├── view_mode_selector.xml
│   │   │   ├── view_scale.xml
│   │   │   └── view_measurement_overlay.xml
│   │   │
│   │   ├── menu/
│   │   │   ├── menu_main.xml
│   │   │   ├── menu_patient.xml
│   │   │   └── menu_settings.xml
│   │   │
│   │   ├── mipmap-mdpi/
│   │   │   ├── ic_launcher.png        # 48x48
│   │   │   └── ic_launcher_round.png  # 48x48
│   │   │
│   │   ├── mipmap-hdpi/
│   │   │   ├── ic_launcher.png        # 72x72
│   │   │   └── ic_launcher_round.png  # 72x72
│   │   │
│   │   ├── mipmap-xhdpi/
│   │   │   ├── ic_launcher.png        # 96x96
│   │   │   └── ic_launcher_round.png  # 96x96
│   │   │
│   │   ├── mipmap-xxhdpi/
│   │   │   ├── ic_launcher.png        # 144x144
│   │   │   └── ic_launcher_round.png  # 144x144
│   │   │
│   │   ├── mipmap-xxxhdpi/
│   │   │   ├── ic_launcher.png        # 192x192
│   │   │   └── ic_launcher_round.png  # 192x192
│   │   │
│   │   ├── values/
│   │   │   ├── strings.xml
│   │   │   ├── strings_es.xml
│   │   │   ├── strings_en.xml
│   │   │   ├── strings_pt.xml
│   │   │   ├── colors.xml
│   │   │   ├── styles.xml
│   │   │   ├── themes.xml
│   │   │   ├── dimens.xml
│   │   │   ├── integers.xml
│   │   │   └── arrays.xml
│   │   │
│   │   ├── values-v21/
│   │   │   ├── styles.xml
│   │   │   └── themes.xml
│   │   │
│   │   ├── values-v23/
│   │   │   ├── styles.xml
│   │   │   └── themes.xml
│   │   │
│   │   └── xml/
│   │       ├── network_security_config.xml
│   │       ├── backup_rules.xml
│   │       └── file_paths.xml
│   │
│   ├── assets/                         # Assets
│   │   ├── examination.json            # Config examenes (modificado)
│   │   ├── examination_vetus.json      # Config veterinaria
│   │   ├── probes.json                 # Config probes
│   │   ├── splash/
│   │   │   ├── logo.svg                # Logo ByteC SVG
│   │   │   ├── animation.json          # Lottie splash
│   │   │   └── background.svg          # Background splash
│   │   ├── templates/
│   │   │   ├── report_template.html    # Template reporte
│   │   │   └── report_template_cn.html
│   │   ├── js/
│   │   │   ├── jsonComm.js             # JS Bridge
│   │   │   └── html2canvas.js          # Canvas capture
│   │   └── fonts/
│   │       ├── Roboto-Regular.ttf
│   │       ├── Roboto-Bold.ttf
│   │       └── Roboto-Medium.ttf
│   │
│   └── lib/                            # Native libraries
│       ├── arm64-v8a/
│       │   ├── libvessel_flow.so       # Vessel analysis
│       │   ├── librs.dscenhance.so     # DSC enhancement
│       │   ├── librs.imageenhance.so   # Image enhancement
│       │   ├── librs.imageexenhancer.so
│       │   ├── librs.imageexenhancer_full.so
│       │   ├── librs.imageexenhancer_v1.so
│       │   ├── libRSSupport.so         # RenderScript
│       │   ├── librsjni.so
│       │   └── librsjni_androidx.so
│       ├── armeabi-v7a/
│       │   └── (mismos .so)
│       ├── x86_64/
│       │   └── (mismos .so)
│       └── x86/
│           └── (mismos .so)
│
├── tools/                              # Herramientas
│   ├── apktool_2.7.0.jar
│   ├──uber-apk-signer.jar
│   └── zipalign
│
├── signing/                            # Certificados
│   ├── bytec.keystore
│   └── bytec_cert.pem
│
├── scripts/                            # Scripts
│   ├── build.sh                        # Build completo
│   ├── decompile.sh                    # Decompilar
│   ├── sign.sh                         # Firmar
│   ├── install.sh                      # Instalar
│   └── clean.sh                        # Limpiar
│
├── docs/                               # Documentacion
│   ├── ARCHITECTURE.md
│   ├── DICOM_INTEGRATION.md
│   ├── PROBE_COMPATIBILITY.md
│   ├── IMAGING_MODES.md
│   ├── NETWORK_PROTOCOL.md
│   ├── SECURITY.md
│   ├── ANALYSIS.md
│   └── CHANGELOG.md
│
├── tests/                              # Tests
│   ├── test_dicom.sh
│   ├── test_connection.sh
│   └── test_ui.sh
│
├── output/                             # APKs generados
│   ├── ByteCUSPro-debug.apk
│   └── ByteCUSPro-release.apk
│
├── README.md
├── LICENSE
├── CHANGELOG.md
├── BYTEC_US_PRO_FULL_SPEC.md           # Este archivo
└── build.sh                            # Script maestro
```

---

# 4. MODULO: SPLASH SCREEN

## 4.1 SplashActivity.java (Codigo Completo)

```java
// Archivo: smali/com/bytec/uspro/SplashActivity.smali
// Equivalente Java para referencia:

package com.bytec.uspro;

import android.animation.AnimatorSet;
import android.animation.ObjectAnimator;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.view.WindowManager;
import android.view.animation.AccelerateDecelerateInterpolator;
import android.view.animation.DecelerateInterpolator;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

public class SplashActivity extends AppCompatActivity {

    // Duraciones de animacion (ms)
    private static final int FADE_DURATION = 800;
    private static final int SCALE_DURATION = 1000;
    private static final int TEXT_DELAY = 500;
    private static final int TEXT_DURATION = 600;
    private static final int VERSION_DELAY = 800;
    private static final int SPLASH_TOTAL_DURATION = 3000;

    // Views
    private ImageView logoImageView;
    private TextView companyNameTextView;
    private TextView versionTextView;
    private TextView taglineTextView;
    private View loadingIndicator;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Full screen immersive mode
        getWindow().setFlags(
            WindowManager.LayoutParams.FLAG_FULLSCREEN,
            WindowManager.LayoutParams.FLAG_FULLSCREEN
        );

        // Ocultar status bar y navigation bar
        getWindow().getDecorView().setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_FULLSCREEN
            | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
        );

        setContentView(R.layout.activity_splash);

        // Inicializar views
        initViews();

        // Ejecutar animaciones
        startAnimations();

        // Navegar a MainActivity
        navigateToMain();
    }

    private void initViews() {
        logoImageView = findViewById(R.id.splash_logo);
        companyNameTextView = findViewById(R.id.splash_company_name);
        versionTextView = findViewById(R.id.splash_version);
        taglineTextView = findViewById(R.id.splash_tagline);
        loadingIndicator = findViewById(R.id.splash_loading);

        // Estado inicial: todo invisible
        logoImageView.setAlpha(0f);
        logoImageView.setScaleX(0.5f);
        logoImageView.setScaleY(0.5f);

        companyNameTextView.setVisibility(View.INVISIBLE);
        versionTextView.setVisibility(View.INVISIBLE);
        taglineTextView.setVisibility(View.INVISIBLE);
    }

    private void startAnimations() {
        // 1. Logo: fade in + scale up
        ObjectAnimator fadeIn = ObjectAnimator.ofFloat(
            logoImageView, "alpha", 0f, 1f);
        fadeIn.setDuration(FADE_DURATION);

        ObjectAnimator scaleX = ObjectAnimator.ofFloat(
            logoImageView, "scaleX", 0.5f, 1f);
        scaleX.setDuration(SCALE_DURATION);

        ObjectAnimator scaleY = ObjectAnimator.ofFloat(
            logoImageView, "scaleY", 0.5f, 1f);
        scaleY.setDuration(SCALE_DURATION);

        AnimatorSet logoAnimSet = new AnimatorSet();
        logoAnimSet.playTogether(fadeIn, scaleX, scaleY);
        logoAnimSet.setInterpolator(new DecelerateInterpolator());
        logoAnimSet.start();

        // 2. Company name: slide up + fade in (despues de delay)
        new Handler().postDelayed(() -> {
            companyNameTextView.setVisibility(View.VISIBLE);
            companyNameTextView.setTranslationY(50);
            companyNameTextView.setAlpha(0);

            ObjectAnimator nameFadeIn = ObjectAnimator.ofFloat(
                companyNameTextView, "alpha", 0f, 1f);
            nameFadeIn.setDuration(TEXT_DURATION);

            ObjectAnimator nameSlideUp = ObjectAnimator.ofFloat(
                companyNameTextView, "translationY", 50, 0);
            nameSlideUp.setDuration(TEXT_DURATION);

            AnimatorSet nameSet = new AnimatorSet();
            nameSet.playTogether(nameFadeIn, nameSlideUp);
            nameSet.setInterpolator(new AccelerateDecelerateInterpolator());
            nameSet.start();
        }, TEXT_DELAY);

        // 3. Version: fade in
        new Handler().postDelayed(() -> {
            versionTextView.setVisibility(View.VISIBLE);
            versionTextView.setAlpha(0);

            ObjectAnimator versionFade = ObjectAnimator.ofFloat(
                versionTextView, "alpha", 0f, 1f);
            versionFade.setDuration(400);
            versionFade.start();
        }, VERSION_DELAY);

        // 4. Tagline: fade in
        new Handler().postDelayed(() -> {
            taglineTextView.setVisibility(View.VISIBLE);
            taglineTextView.setAlpha(0);

            ObjectAnimator tagFade = ObjectAnimator.ofFloat(
                taglineTextView, "alpha", 0f, 1f);
            tagFade.setDuration(400);
            tagFade.start();
        }, VERSION_DELAY + 200);
    }

    private void navigateToMain() {
        new Handler().postDelayed(() -> {
            Intent intent = new Intent(SplashActivity.this, MainActivity.class);
            startActivity(intent);

            // Transicion personalizada
            overridePendingTransition(
                android.R.anim.fade_in,
                android.R.anim.fade_out
            );

            finish();
        }, SPLASH_TOTAL_DURATION);
    }

    @Override
    public void onBackPressed() {
        // Bloquear back en splash
        // No hacer nada
    }
}
```

## 4.2 activity_splash.xml (Layout Completo)

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/bg_splash_gradient"
    android:gravity="center">

    <!-- Background gradient effect -->
    <View
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/bg_splash_gradient"/>

    <!-- Logo Container -->
    <RelativeLayout
        android:id="@+id/splash_logo_container"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true">

        <!-- Glow effect behind logo -->
        <View
            android:id="@+id/splash_glow"
            android:layout_width="220dp"
            android:layout_height="220dp"
            android:layout_centerInParent="true"
            android:background="@drawable/bg_glow"/>

        <!-- Logo -->
        <ImageView
            android:id="@+id/splash_logo"
            android:layout_width="180dp"
            android:layout_height="180dp"
            android:layout_centerInParent="true"
            android:src="@drawable/bytec_logo_circle"
            android:scaleType="fitCenter"
            android:elevation="8dp"/>

    </RelativeLayout>

    <!-- Text Container -->
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/splash_logo_container"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="30dp"
        android:orientation="vertical"
        android:gravity="center">

        <!-- Company Name -->
        <TextView
            android:id="@+id/splash_company_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ByteC"
            android:textColor="@color/bytec_text_primary"
            android:textSize="36sp"
            android:textStyle="bold"
            android:fontFamily="@font/roboto_bold"
            android:letterSpacing="0.15"
            android:visibility="invisible"/>

        <!-- Tagline -->
        <TextView
            android:id="@+id/splash_tagline"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="Medical Devices"
            android:textColor="@color/bytec_text_secondary"
            android:textSize="16sp"
            android:textStyle="normal"
            android:fontFamily="@font/roboto"
            android:letterSpacing="0.3"
            android:visibility="invisible"/>

    </LinearLayout>

    <!-- Version and Loading -->
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="60dp"
        android:orientation="vertical"
        android:gravity="center">

        <!-- Version -->
        <TextView
            android:id="@+id/splash_version"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="US Pro v2.0.0"
            android:textColor="@color/bytec_text_hint"
            android:textSize="12sp"
            android:fontFamily="@font/roboto"
            android:visibility="invisible"/>

        <!-- Loading indicator -->
        <ProgressBar
            android:id="@+id/splash_loading"
            style="?android:attr/progressBarStyleSmall"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_marginTop="16dp"
            android:indeterminateTint="@color/bytec_accent"/>

    </LinearLayout>

    <!-- Copyright -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="20dp"
        android:text="© 2024 ByteC Medical Devices"
        android:textColor="@color/bytec_text_hint"
        android:textSize="10sp"
        android:fontFamily="@font/roboto"/>

</RelativeLayout>
```

## 4.3 bg_splash_gradient.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <gradient
        android:type="radial"
        android:gradientRadius="400"
        android:centerX="0.5"
        android:centerY="0.4"
        android:startColor="#1a1a2e"
        android:centerColor="#16213e"
        android:endColor="#0f0f23"/>
</shape>
```

## 4.4 bg_glow.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <gradient
        android:type="radial"
        android:gradientRadius="110"
        android:startColor="#1E88E533"
        android:endColor="#00000000"/>
</shape>
```

## 4.5 bytec_logo_circle.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="180dp"
    android:height="180dp"
    android:viewportWidth="180"
    android:viewportHeight="180">

    <!-- Background circle -->
    <path
        android:fillColor="#1E88E5"
        android:pathData="M90,90m-85,0a85,85 0,1 1,170 0a85,85 0,1 1,-170 0"/>

    <!-- Inner circle -->
    <path
        android:fillColor="#1565C0"
        android:pathData="M90,90m-70,0a70,70 0,1 1,140 0a70,70 0,1 1,-140 0"/>

    <!-- B letter -->
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M65,50 L65,130 L85,130 Q105,130 105,115 Q105,100 95,95
        Q108,90 108,75 Q108,50 85,50 Z M80,65 L80,85 Q85,85 85,90
        Q85,95 80,95 L80,65 Z M80,100 L85,100 Q95,100 95,110
        Q95,120 85,120 L80,120 Z"/>

    <!-- ByteC text -->
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M55,138 L55,148 L115,148 L115,138 Z"/>
    <path
        android:fillColor="#00E676"
        android:pathData="M55,150 L55,155 L115,155 L115,150 Z"/>

</vector>
```

## 4.6 Animaciones XML

### fade_in.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator">
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="800"/>
</set>
```

### fade_out.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_interpolator">
    <alpha
        android:fromAlpha="1.0"
        android:toAlpha="0.0"
        android:duration="500"/>
</set>
```

### slide_up.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator">
    <translate
        android:fromYDelta="100%"
        android:toYDelta="0%"
        android:duration="600"/>
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="600"/>
</set>
```

### scale_in.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/overshoot_interpolator">
    <scale
        android:fromXScale="0.3"
        android:toXScale="1.0"
        android:fromYScale="0.3"
        android:toYScale="1.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:duration="800"/>
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="800"/>
</set>
```

---

# 5. MODULO: REBRAND COMPLETO

## 5.1 AndroidManifest.xml (Completo)

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.bytec.uspro"
    android:versionCode="2"
    android:versionName="2.0.0"
    android:installLocation="auto"
    android:hardwareAccelerated="true">

    <!-- Permisos -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.USB_PERMISSION"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.NFC"/>

    <!-- Hardware features -->
    <uses-feature android:name="android.hardware.usb.host" android:required="false"/>
    <uses-feature android:name="android.hardware.wifi" android:required="false"/>
    <uses-feature android:name="android.hardware.camera" android:required="false"/>
    <uses-feature android:name="android.hardware.screen.landscape" android:required="true"/>

    <!-- Min SDK -->
    <uses-sdk android:minSdkVersion="23" android:targetSdkVersion="33"/>

    <!-- Application -->
    <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:networkSecurityConfig="@xml/network_security_config"
        android:usesCleartextTraffic="true"
        android:largeHeap="true">

        <!-- Splash Activity (Launcher) -->
        <activity
            android:name=".SplashActivity"
            android:exported="true"
            android:screenOrientation="landscape"
            android:theme="@style/SplashTheme"
            android:windowSoftInputMode="adjustPan">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

        <!-- Main Activity -->
        <activity
            android:name=".MainActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:theme="@style/AppTheme"
            android:windowSoftInputMode="adjustResize"
            android:launchMode="singleTop"/>

        <!-- Patient Activity -->
        <activity
            android:name=".PatientActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- Report Activity -->
        <activity
            android:name=".ReportActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- Settings Activity -->
        <activity
            android:name=".SettingActivity"
            android:exported="false"
            android:screenOrientation="portrait"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- DICOM Activity -->
        <activity
            android:name=".DicomActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- Worklist Activity -->
        <activity
            android:name=".WorklistActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- Gallery Activity -->
        <activity
            android:name=".GalleryActivity"
            android:exported="false"
            android:screenOrientation="landscape"
            android:theme="@style/AppTheme"
            android:parentActivityName=".MainActivity"/>

        <!-- File Provider -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"/>
        </provider>

        <!-- Broadcast Receiver for USB -->
        <receiver
            android:name=".UsbReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.bytec.uspro.USB_PERMISSION"/>
                <action android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED"/>
            </intent-filter>
        </receiver>

        <!-- Boot Receiver -->
        <receiver
            android:name=".BootReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>

        <!-- Service for background DICOM -->
        <service
            android:name=".DicomService"
            android:exported="false"
            android:foregroundServiceType="dataSync"/>

    </application>

</manifest>
```

## 5.2 strings.xml (Completo)

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- App -->
    <string name="app_name">ByteC US Pro</string>
    <string name="company_name">ByteC Medical Devices</string>
    <string name="app_version">v2.0.0</string>
    <string name="copyright">© 2024 ByteC Medical Devices. All rights reserved.</string>

    <!-- Splash -->
    <string name="splash_tagline">Advanced Ultrasound Imaging</string>

    <!-- Navigation -->
    <string name="nav_home">Home</string>
    <string name="nav_scan">Scan</string>
    <string name="nav_patients">Patients</string>
    <string name="nav_gallery">Gallery</string>
    <string name="nav_settings">Settings</string>
    <string name="nav_help">Help</string>

    <!-- Modes -->
    <string name="mode_b">B-Mode</string>
    <string name="mode_m">M-Mode</string>
    <string name="mode_color">Color</string>
    <string name="mode_pw">PW</string>
    <string name="mode_pw_ext">PW Ext</string>
    <string name="mode_biopsy">Biopsy</string>
    <string name="mode_dual">Dual</string>

    <!-- Controls -->
    <string name="btn_freeze">Freeze</string>
    <string name="btn_unfreeze">Unfreeze</string>
    <string name="btn_capture">Capture</string>
    <string name="btn_measure">Measure</string>
    <string name="btn_annotate">Annotate</string>
    <string name="btn_export">Export</string>
    <string name="btn_report">Report</string>
    <string name="btn_dicom">DICOM</string>
    <string name="btn_worklist">Worklist</string>
    <string name="btn_probe">Probe</string>
    <string name="btn_settings">Settings</string>
    <string name="btn_back">Back</string>
    <string name="btn_save">Save</string>
    <string name="btn_cancel">Cancel</string>
    <string name="btn_delete">Delete</string>
    <string name="btn_edit">Edit</string>
    <string name="btn_add">Add</string>
    <string name="btn_search">Search</string>
    <string name="btn_send">Send</string>
    <string name="btn_connect">Connect</string>
    <string name="btn_disconnect">Disconnect</string>
    <string name="btn_refresh">Refresh</string>
    <string name="btn_apply">Apply</string>
    <string name="btn_reset">Reset</string>

    <!-- Parameters -->
    <string name="param_gain">Gain</string>
    <string name="param_depth">Depth</string>
    <string name="param_focus">Focus</string>
    <string name="param_dr">Dynamic Range</string>
    <string name="param_enhance">Enhancement</string>
    <string name="param_harmonic">Harmonic</string>
    <string name="param_zoom">Zoom</string>
    <string name="param_compound">Compound</string>
    <string name="param_power">Power</string>

    <!-- Color Flow -->
    <string name="color_gain">Color Gain</string>
    <string name="color_wall_filter">Wall Filter</string>
    <string name="color_prf">PRF</string>
    <string name="color_angle">Angle</string>
    <string name="color_baseline">Baseline</string>

    <!-- Measurements -->
    <string name="measure_distance">Distance</string>
    <string name="measure_area">Area</string>
    <string name="measure_volume">Volume</string>
    <string name="measure_angle">Angle</string>
    <string name="measure_trace">Trace</string>
    <string name="measure_velocity">Velocity</string>
    <string name="measure_heart_rate">Heart Rate</string>
    <string name="measure_ti">Thermal Index</string>
    <string name="measure_mechanical_index">Mechanical Index</string>

    <!-- Patient -->
    <string name="patient_name">Patient Name</string>
    <string name="patient_id">Patient ID</string>
    <string name="patient_birth_date">Birth Date</string>
    <string name="patient_sex">Sex</string>
    <string name="patient_sex_male">Male</string>
    <string name="patient_sex_female">Female</string>
    <string name="patient_sex_other">Other</string>
    <string name="patient_phone">Phone</string>
    <string name="patient_email">Email</string>
    <string name="patient_notes">Notes</string>
    <string name="patient_referring">Referring Physician</string>
    <string name="patient_accession">Accession Number</string>

    <!-- Studies -->
    <string name="study_date">Study Date</string>
    <string name="study_time">Study Time</string>
    <string name="study_description">Study Description</string>
    <string name="study_series">Series</string>
    <string name="study_images">Images</string>
    <string name="study_modality">Modality</string>

    <!-- Examinations -->
    <string name="exam_thyroid">Thyroid</string>
    <string name="exam_small_parts">Small Parts</string>
    <string name="exam_pediatrics">Pediatrics</string>
    <string name="exam_vascular">Vascular</string>
    <string name="exam_carotid">Carotid</string>
    <string name="exam_breast">Breast</string>
    <string name="exam_msk">MSK</string>
    <string name="exam_nerve">Nerve</string>
    <string name="exam_abdomen">Abdomen</string>
    <string name="exam_gynecology">Gynecology</string>
    <string name="exam_obstetric">Obstetric</string>
    <string name="exam_cardiac">Cardiac</string>
    <string name="exam_urology">Urology</string>
    <string name="exam_kidney">Kidney</string>
    <string name="exam_lung">Lung</string>
    <string name="exam_eye">Eye</string>
    <string name="exam_vessel_flow">Vessel Flow</string>

    <!-- Probes -->
    <string name="probe_linear">Linear Probe</string>
    <string name="probe_convex">Convex Probe</string>
    <string name="probe_phased">Phased Probe</string>
    <string name="probe_select">Select Probe</string>
    <string name="probe_connecting">Connecting to probe...</string>
    <string name="probe_connected">Probe connected</string>
    <string name="probe_disconnected">Probe disconnected</string>
    <string name="probe_error">Probe connection error</string>

    <!-- DICOM -->
    <string name="dicom_title">DICOM Configuration</string>
    <string name="dicom_local_ae">Local AE Title</string>
    <string name="dicom_remote_ae">Remote AE Title</string>
    <string name="dicom_host">PACS Host</string>
    <string name="dicom_port">DICOM Port</string>
    <string name="dicom_mwl_ae">MWL AE Title</string>
    <string name="dicom_mwl_host">MWL Host</string>
    <string name="dicom_mwl_port">MWL Port</string>
    <string name="dicom_sending">Sending to PACS...</string>
    <string name="dicom_sent">Image sent successfully</string>
    <string name="dicom_failed">Failed to send image</string>
    <string name="dicom_echo_test">Echo Test</string>
    <string name="dicom_echo_success">Connection OK</string>
    <string name="dicom_echo_failed">Connection Failed</string>

    <!-- Worklist -->
    <string name="worklist_title">Modality Worklist</string>
    <string name="worklist_search">Search Patient</string>
    <string name="worklist_no_results">No results found</string>
    <string name="worklist_select">Select from Worklist</string>

    <!-- Settings -->
    <string name="settings_title">Settings</string>
    <string name="settings_general">General</string>
    <string name="settings_display">Display</string>
    <string name="settings_sound">Sound</string>
    <string name="settings_storage">Storage</string>
    <string name="settings_network">Network</string>
    <string name="settings_about">About</string>
    <string name="settings_language">Language</string>
    <string name="settings_theme">Theme</string>
    <string name="settings_auto_save">Auto Save</string>
    <string name="settings_show_scale">Show Scale</string>
    <string name="settings_show_annotations">Show Annotations</string>
    <string name="settings_capture_sound">Capture Sound</string>
    <string name="settings_storage_path">Storage Path</string>
    <string name="settings_max_images">Max Images</string>

    <!-- Messages -->
    <string name="msg_connecting">Connecting to probe...</string>
    <string name="msg_connected">Connected successfully</string>
    <string name="msg_disconnected">Disconnected</string>
    <string name="msg_error">Error occurred</string>
    <string name="msg_saving">Saving...</string>
    <string name="msg_saved">Saved successfully</string>
    <string name="msg_deleted">Deleted successfully</string>
    <string name="msg_confirm_delete">Are you sure you want to delete?</string>
    <string name="msg_no_patient">No patient selected</string>
    <string name="msg_probe_required">Probe connection required</string>
    <string name="msg_permission_required">Permission required</string>
    <string name="msg_storage_full">Storage full</string>
    <string name="msg_export_success">Export completed</string>
    <string name="msg_export_failed">Export failed</string>

    <!-- Units -->
    <string name="unit_mm">mm</string>
    <string name="unit_cm">cm</string>
    <string name="unit_m">m</string>
    <string name="unit_mhz">MHz</string>
    <string name="unit_db">dB</string>
    <string name="unit_khz">kHz</string>
    <string name="unit_mps">m/s</string>
    <string name="unit_cms">cm/s</string>
    <string name="unit_bpm">bpm</string>
    <string name="unit_deg">°</string>
    <string name="unit_cm2">cm²</string>
    <string name="unit_cm3">cm³</string>

    <!-- Dialogs -->
    <string name="dialog_confirm">Confirm</string>
    <string name="dialog_cancel">Cancel</string>
    <string name="dialog_ok">OK</string>
    <string name="dialog_yes">Yes</string>
    <string name="dialog_no">No</string>

    <!-- Errors -->
    <string name="error_connection">Connection error</string>
    <string name="error_timeout">Connection timeout</string>
    <string name="error_invalid_data">Invalid data</string>
    <string name="error_file_not_found">File not found</string>
    <string name="error_permission_denied">Permission denied</string>
    <string name="error_unknown">Unknown error</string>

</resources>
```

## 5.3 strings_es.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">ByteC US Pro</string>
    <string name="splash_tagline">Imagen por Ultrasonido Avanzada</string>
    <string name="btn_freeze">Congelar</string>
    <string name="btn_unfreeze">Descongelar</string>
    <string name="btn_capture">Capturar</string>
    <string name="btn_measure">Medir</string>
    <string name="btn_export">Exportar</string>
    <string name="btn_report">Reporte</string>
    <string name="btn_connect">Conectar</string>
    <string name="btn_disconnect">Desconectar</string>
    <string name="probe_connecting">Conectando al probe...</string>
    <string name="probe_connected">Probe conectado</string>
    <string name="probe_disconnected">Probe desconectado</string>
    <string name="param_gain">Ganancia</string>
    <string name="param_depth">Profundidad</string>
    <string name="param_focus">Enfoque</string>
    <string name="patient_name">Nombre del Paciente</string>
    <string name="patient_id">ID del Paciente</string>
    <string name="settings_title">Configuracion</string>
    <string name="msg_saving">Guardando...</string>
    <string name="msg_saved">Guardado exitosamente</string>
</resources>
```

## 5.4 colors.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- ByteC Brand Colors -->
    <color name="bytec_primary">#1E88E5</color>
    <color name="bytec_primary_dark">#1565C0</color>
    <color name="bytec_primary_light">#42A5F5</color>
    <color name="bytec_accent">#00E676</color>
    <color name="bytec_accent_dark">#00C853</color>
    <color name="bytec_warning">#FFD740</color>
    <color name="bytec_error">#FF5252</color>
    <color name="bytec_success">#00E676</color>

    <!-- UI Colors -->
    <color name="bytec_background">#121212</color>
    <color name="bytec_background_light">#1E1E1E</color>
    <color name="bytec_surface">#252525</color>
    <color name="bytec_surface_light">#2D2D2D</color>
    <color name="bytec_card">#1A1A1A</color>

    <!-- Text Colors -->
    <color name="bytec_text_primary">#FFFFFF</color>
    <color name="bytec_text_secondary">#B0B0B0</color>
    <color name="bytec_text_hint">#6B6B6B</color>
    <color name="bytec_text_disabled">#4A4A4A</color>

    <!-- Status Colors -->
    <color name="status_connected">#00E676</color>
    <color name="status_disconnected">#FF5252</color>
    <color name="status_frozen">#FFD740</color>
    <color name="status_processing">#42A5F5</color>
    <color name="status_warning">#FF9100</color>

    <!-- Medical UI Colors -->
    <color name="ultrasound_background">#000000</color>
    <color name="ultrasound_grayscale_high">#FFFFFF</color>
    <color name="ultrasound_grayscale_low">#000000</color>

    <!-- Doppler Colors -->
    <color name="doppler_red_positive">#FF1744</color>
    <color name="doppler_red">#D50000</color>
    <color name="doppler_blue_negative">#2979FF</color>
    <color name="doppler_blue">#0D47A1</color>
    <color name="doppler_yellow">#FFD600</color>
    <color name="doppler_turquoise">#00BCD4</color>

    <!-- PW Colors -->
    <color name="pw_trace">#00E676</color>
    <color name="pw_baseline">#555555</color>
    <color name="pw_envelope">#00E676</color>

    <!-- Measurement Colors -->
    <color name="measure_distance">#FF9100</color>
    <color name="measure_area">#E040FB</color>
    <color name="measure_angle">#FFEB3B</color>
    <color name="measure_trace">#00BCD4</color>

    <!-- Biopsy Guide -->
    <color name="biopsy_guide">#FFEB3B</color>
    <color name="biopsy_safe">#00E676</color>
    <color name="biopsy_danger">#FF5252</color>

    <!-- Scale -->
    <color name="scale_text">#FFFFFF</color>
    <color name="scale_line">#555555</color>

    <!-- Divider -->
    <color name="divider">#333333</color>

    <!-- Ripple -->
    <color name="ripple">#33FFFFFF</color>
</resources>
```

## 5.5 themes.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- Splash Theme -->
    <style name="SplashTheme" parent="Theme.AppCompat.NoActionBar">
        <item name="android:windowBackground">@drawable/bg_splash_gradient</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowAnimationStyle">@null</item>
        <item name="colorPrimary">@color/bytec_primary</item>
        <item name="colorPrimaryDark">@color/bytec_primary_dark</item>
        <item name="colorAccent">@color/bytec_accent</item>
    </style>

    <!-- App Theme -->
    <style name="AppTheme" parent="Theme.AppCompat.DayNight.NoActionBar">
        <item name="colorPrimary">@color/bytec_primary</item>
        <item name="colorPrimaryDark">@color/bytec_primary_dark</item>
        <item name="colorAccent">@color/bytec_accent</item>
        <item name="android:windowBackground">@color/bytec_background</item>
        <item name="android:textColorPrimary">@color/bytec_text_primary</item>
        <item name="android:textColorSecondary">@color/bytec_text_secondary</item>
        <item name="android:textColorHint">@color/bytec_text_hint</item>
        <item name="android:statusBarColor">@color/bytec_background</item>
        <item name="android:navigationBarColor">@color/bytec_background</item>
        <item name="android:windowLightStatusBar">false</item>
    </style>

    <!-- Button Styles -->
    <style name="ByteCButton" parent="Widget.AppCompat.Button.Colored">
        <item name="backgroundTint">@color/bytec_primary</item>
        <item name="android:textColor">@color/bytec_text_primary</item>
        <item name="cornerRadius">8dp</item>
        <item name="android:padding">12dp</item>
        <item name="android:textAllCaps">false</item>
    </style>

    <style name="ByteCButtonSecondary">
        <item name="backgroundTint">@color/bytec_surface</item>
        <item name="android:textColor">@color/bytec_text_primary</item>
        <item name="cornerRadius">8dp</item>
    </style>

    <style name="ByteCButtonAccent">
        <item name="backgroundTint">@color/bytec_accent</item>
        <item name="android:textColor">@color/bytec_background</item>
        <item name="cornerRadius">8dp</item>
    </style>

    <style name="ByteCButtonDanger">
        <item name="backgroundTint">@color/bytec_error</item>
        <item name="android:textColor">@color/bytec_text_primary</item>
        <item name="cornerRadius">8dp</item>
    </style>

    <!-- Card Style -->
    <style name="ByteCCard" parent="Widget.AppCompat.CardView">
        <item name="cardBackgroundColor">@color/bytec_card</item>
        <item name="cardCornerRadius">12dp</item>
        <item name="cardElevation">4dp</item>
        <item name="contentPadding">16dp</item>
    </style>

    <!-- Input Style -->
    <style name="ByteCInput" parent="Widget.AppCompat.EditText">
        <item name="android:background">@drawable/bg_input</item>
        <item name="android:textColor">@color/bytec_text_primary</item>
        <item name="android:textColorHint">@color/bytec_text_hint</item>
        <item name="android:padding">12dp</item>
    </style>

    <!-- Dialog Style -->
    <style name="ByteCDialog" parent="Theme.AppCompat.Dialog">
        <item name="android:windowBackground">@color/bytec_surface</item>
        <item name="android:textColorPrimary">@color/bytec_text_primary</item>
        <item name="android:textColorSecondary">@color/bytec_text_secondary</item>
    </style>

</resources>
```

## 5.6 dimens.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Spacing -->
    <dimen name="spacing_xs">4dp</dimen>
    <dimen name="spacing_sm">8dp</dimen>
    <dimen name="spacing_md">16dp</dimen>
    <dimen name="spacing_lg">24dp</dimen>
    <dimen name="spacing_xl">32dp</dimen>
    <dimen name="spacing_xxl">48dp</dimen>

    <!-- Text Sizes -->
    <dimen name="text_xs">10sp</dimen>
    <dimen name="text_sm">12sp</dimen>
    <dimen name="text_md">14sp</dimen>
    <dimen name="text_lg">16sp</dimen>
    <dimen name="text_xl">20sp</dimen>
    <dimen name="text_xxl">24sp</dimen>
    <dimen name="text_xxxl">32sp</dimen>

    <!-- Corner Radius -->
    <dimen name="radius_sm">4dp</dimen>
    <dimen name="radius_md">8dp</dimen>
    <dimen name="radius_lg">12dp</dimen>
    <dimen name="radius_xl">16dp</dimen>
    <dimen name="radius_full">9999dp</dimen>

    <!-- Elevation -->
    <dimen name="elevation_sm">2dp</dimen>
    <dimen name="elevation_md">4dp</dimen>
    <dimen name="elevation_lg">8dp</dimen>

    <!-- Icon Sizes -->
    <dimen name="icon_sm">16dp</dimen>
    <dimen name="icon_md">24dp</dimen>
    <dimen name="icon_lg">32dp</dimen>
    <dimen name="icon_xl">48dp</dimen>

    <!-- Button Heights -->
    <dimen name="button_height_sm">32dp</dimen>
    <dimen name="button_height_md">40dp</dimen>
    <dimen name="button_height_lg">48dp</dimen>

    <!-- Splash -->
    <dimen name="splash_logo_size">180dp</dimen>
    <dimen name="splash_text_size">36sp</dimen>
</resources>
```

## 5.7 network_security_config.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>

    <!-- DICOM servers -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">192.168.0.0/16</domain>
        <domain includeSubdomains="true">10.0.0.0/8</domain>
        <domain includeSubdomains="true">172.16.0.0/12</domain>
    </domain-config>

    <!-- Cloud API (HTTPS only) -->
    <domain-config>
        <domain includeSubdomains="true">api.bytecmedical.com</domain>
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

---

# 6. MODULO: UI MODERNA

## 6.1 activity_main.xml (Layout Principal)

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/bytec_background">

    <!-- Top Bar -->
    <LinearLayout
        android:id="@+id/top_bar"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:layout_alignParentTop="true"
        android:background="@color/bytec_surface"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="8dp">

        <!-- Logo -->
        <ImageView
            android:layout_width="32dp"
            android:layout_height="32dp"
            android:src="@drawable/bytec_logo_circle"
            android:layout_marginEnd="8dp"/>

        <!-- App name -->
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="ByteC US Pro"
            android:textColor="@color/bytec_text_primary"
            android:textSize="16sp"
            android:textStyle="bold"/>

        <!-- Probe Status -->
        <LinearLayout
            android:id="@+id/probe_status"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical"
            android:layout_marginEnd="16dp">

            <View
                android:id="@+id/probe_indicator"
                android:layout_width="8dp"
                android:layout_height="8dp"
                android:background="@drawable/bg_circle"
                android:backgroundTint="@color/status_disconnected"
                android:layout_marginEnd="4dp"/>

            <TextView
                android:id="@+id/probe_status_text"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Disconnected"
                android:textColor="@color/bytec_text_secondary"
                android:textSize="12sp"/>
        </LinearLayout>

        <!-- Patient Info -->
        <TextView
            android:id="@+id/patient_info"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="No Patient"
            android:textColor="@color/bytec_text_secondary"
            android:textSize="12sp"
            android:layout_marginEnd="16dp"/>

        <!-- Settings Button -->
        <ImageButton
            android:id="@+id/btn_settings"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:src="@drawable/ic_settings"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="Settings"/>

    </LinearLayout>

    <!-- Main Content -->
    <LinearLayout
        android:id="@+id/main_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/top_bar"
        android:layout_above="@id/bottom_bar"
        android:orientation="horizontal">

        <!-- Left Panel - Mode Selector -->
        <LinearLayout
            android:id="@+id/left_panel"
            android:layout_width="60dp"
            android:layout_height="match_parent"
            android:background="@color/bytec_surface"
            android:orientation="vertical"
            android:gravity="center"
            android:paddingVertical="8dp">

            <ImageButton
                android:id="@+id/mode_b"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_mode_b"
                android:background="@drawable/bg_mode_button"
                android:layout_marginBottom="8dp"
                android:contentDescription="B-Mode"/>

            <ImageButton
                android:id="@+id/mode_m"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_mode_m"
                android:background="@drawable/bg_mode_button"
                android:layout_marginBottom="8dp"
                android:contentDescription="M-Mode"/>

            <ImageButton
                android:id="@+id/mode_color"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_mode_color"
                android:background="@drawable/bg_mode_button"
                android:layout_marginBottom="8dp"
                android:contentDescription="Color Doppler"/>

            <ImageButton
                android:id="@+id/mode_pw"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_mode_pw"
                android:background="@drawable/bg_mode_button"
                android:layout_marginBottom="8dp"
                android:contentDescription="PW Doppler"/>

            <ImageButton
                android:id="@+id/mode_biopsy"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_biopsy"
                android:background="@drawable/bg_mode_button"
                android:contentDescription="Biopsy Guide"/>

        </LinearLayout>

        <!-- Center - Ultrasound View -->
        <FrameLayout
            android:id="@+id/ultrasound_container"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@color/ultrasound_background">

            <!-- B-Mode View -->
            <View
                android:id="@+id/b_mode_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>

            <!-- Scale Overlay -->
            <LinearLayout
                android:id="@+id/scale_overlay"
                android:layout_width="40dp"
                android:layout_height="match_parent"
                android:layout_gravity="end"
                android:orientation="vertical"
                android:padding="4dp">

                <TextView
                    android:id="@+id/scale_depth"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="15cm"
                    android:textColor="@color/scale_text"
                    android:textSize="10sp"/>

            </LinearLayout>

            <!-- Measurement Overlay -->
            <View
                android:id="@+id/measurement_overlay"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:visibility="gone"/>

            <!-- Biopsy Guide Overlay -->
            <View
                android:id="@+id/biopsy_overlay"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:visibility="gone"/>

            <!-- Freeze Indicator -->
            <TextView
                android:id="@+id/freeze_indicator"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="top|start"
                android:layout_margin="8dp"
                android:text="FROZEN"
                android:textColor="@color/status_frozen"
                android:textSize="14sp"
                android:textStyle="bold"
                android:visibility="gone"/>

            <!-- Mode Label -->
            <TextView
                android:id="@+id/mode_label"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="top|end"
                android:layout_margin="8dp"
                android:text="B-Mode"
                android:textColor="@color/bytec_accent"
                android:textSize="12sp"/>

            <!-- FPS Counter -->
            <TextView
                android:id="@+id/fps_counter"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="bottom|start"
                android:layout_margin="8dp"
                android:text="30 fps"
                android:textColor="@color/bytec_text_hint"
                android:textSize="10sp"/>

        </FrameLayout>

        <!-- Right Panel - Controls -->
        <LinearLayout
            android:id="@+id/right_panel"
            android:layout_width="80dp"
            android:layout_height="match_parent"
            android:background="@color/bytec_surface"
            android:orientation="vertical"
            android:gravity="center"
            android:paddingVertical="8dp">

            <!-- Gain -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:gravity="center"
                android:layout_marginBottom="8dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Gain"
                    android:textColor="@color/bytec_text_secondary"
                    android:textSize="10sp"/>

                <SeekBar
                    android:id="@+id/seek_gain"
                    style="@style/ByteCSeekBar"
                    android:layout_width="60dp"
                    android:layout_height="wrap_content"
                    android:max="100"
                    android:progress="80"/>

                <TextView
                    android:id="@+id/value_gain"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="80"
                    android:textColor="@color/bytec_text_primary"
                    android:textSize="12sp"/>
            </LinearLayout>

            <!-- Depth -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:gravity="center"
                android:layout_marginBottom="8dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Depth"
                    android:textColor="@color/bytec_text_secondary"
                    android:textSize="10sp"/>

                <SeekBar
                    android:id="@+id/seek_depth"
                    style="@style/ByteCSeekBar"
                    android:layout_width="60dp"
                    android:layout_height="wrap_content"
                    android:max="100"
                    android:progress="50"/>

                <TextView
                    android:id="@+id/value_depth"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="15cm"
                    android:textColor="@color/bytec_text_primary"
                    android:textSize="12sp"/>
            </LinearLayout>

            <!-- DR -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:gravity="center"
                android:layout_marginBottom="8dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="DR"
                    android:textColor="@color/bytec_text_secondary"
                    android:textSize="10sp"/>

                <SeekBar
                    android:id="@+id/seek_dr"
                    style="@style/ByteCSeekBar"
                    android:layout_width="60dp"
                    android:layout_height="wrap_content"
                    android:max="100"
                    android:progress="70"/>

                <TextView
                    android:id="@+id/value_dr"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="70"
                    android:textColor="@color/bytec_text_primary"
                    android:textSize="12sp"/>
            </LinearLayout>

            <!-- Harmonic Toggle -->
            <ToggleButton
                android:id="@+id/toggle_harmonic"
                android:layout_width="60dp"
                android:layout_height="32dp"
                android:textOn="THI"
                android:textOff="THI"
                android:checked="true"
                android:layout_marginBottom="8dp"/>

            <!-- Focus -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:gravity="center">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Focus"
                    android:textColor="@color/bytec_text_secondary"
                    android:textSize="10sp"/>

                <RadioGroup
                    android:id="@+id/radio_focus"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:orientation="vertical">

                    <RadioButton
                        android:id="@+id/focus_1"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="1"
                        android:checked="true"/>

                    <RadioButton
                        android:id="@+id/focus_2"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="2"/>

                    <RadioButton
                        android:id="@+id/focus_3"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="3"/>

                </RadioGroup>

            </LinearLayout>

        </LinearLayout>

    </LinearLayout>

    <!-- Bottom Bar -->
    <LinearLayout
        android:id="@+id/bottom_bar"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_alignParentBottom="true"
        android:background="@color/bytec_surface"
        android:gravity="center"
        android:orientation="horizontal"
        android:paddingHorizontal="16dp">

        <!-- Patient Button -->
        <LinearLayout
            android:id="@+id/btn_patient"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground"
            android:padding="4dp">

            <ImageView
                android:layout_width="24dp"
                android:layout_height="24dp"
                android:src="@drawable/ic_patient"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Patient"
                android:textColor="@color/bytec_text_secondary"
                android:textSize="10sp"/>
        </LinearLayout>

        <!-- Measure Button -->
        <LinearLayout
            android:id="@+id/btn_measure"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground"
            android:padding="4dp">

            <ImageView
                android:layout_width="24dp"
                android:layout_height="24dp"
                android:src="@drawable/ic_measure"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Measure"
                android:textColor="@color/bytec_text_secondary"
                android:textSize="10sp"/>
        </LinearLayout>

        <!-- Freeze/Capture Button (Large Center) -->
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1.5"
            android:orientation="vertical"
            android:gravity="center"
            android:padding="4dp">

            <!-- Freeze -->
            <ImageButton
                android:id="@+id/btn_freeze"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_freeze"
                android:background="@drawable/bg_circle_button"
                android:layout_marginBottom="2dp"
                android:contentDescription="Freeze"/>

            <!-- Capture (hidden when frozen) -->
            <ImageButton
                android:id="@+id/btn_capture"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:src="@drawable/ic_capture"
                android:background="@drawable/bg_circle_button_accent"
                android:visibility="gone"
                android:layout_marginBottom="2dp"
                android:contentDescription="Capture"/>
        </LinearLayout>

        <!-- DICOM Button -->
        <LinearLayout
            android:id="@+id/btn_dicom"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground"
            android:padding="4dp">

            <ImageView
                android:layout_width="24dp"
                android:layout_height="24dp"
                android:src="@drawable/ic_dicom"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="DICOM"
                android:textColor="@color/bytec_text_secondary"
                android:textSize="10sp"/>
        </LinearLayout>

        <!-- Export Button -->
        <LinearLayout
            android:id="@+id/btn_export"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground"
            android:padding="4dp">

            <ImageView
                android:layout_width="24dp"
                android:layout_height="24dp"
                android:src="@drawable/ic_export"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Export"
                android:textColor="@color/bytec_text_secondary"
                android:textSize="10sp"/>
        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
```

---

# 7. MODULO: CONEXION PROBE

## 7.1 ProbeManager.smali (Logica Completa)

```java
// Referencia Java - ProbeManager.java

package com.bytec.uspro;

import android.os.Handler;
import android.os.Looper;
import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class ProbeManager {

    // Singleton
    private static ProbeManager instance;
    private ProbeManager() {}
    public static synchronized ProbeManager getInstance() {
        if (instance == null) instance = new ProbeManager();
        return instance;
    }

    // Configuracion del probe
    public enum ProbeType {
        LINEAR, CONVEX, PHASED, ENDOCAVITY
    }

    public enum ConnectionType {
        WIFI, USB, NONE
    }

    public enum ProbeState {
        DISCONNECTED, CONNECTING, CONNECTED, STREAMING, ERROR
    }

    // Estado
    private ProbeState state = ProbeState.DISCONNECTED;
    private ConnectionType connectionType = ConnectionType.NONE;
    private ProbeType probeType = ProbeType.LINEAR;
    private String probeId = "";
    private String probeModel = "";
    private String firmwareVersion = "";

    // Conexion
    private Socket socket;
    private OutputStream outputStream;
    private InputStream inputStream;
    private ExecutorService executor;
    private Handler mainHandler;

    // Callbacks
    public interface ProbeCallback {
        void onStateChanged(ProbeState newState);
        void onFrameReceived(byte[] frame, int width, int height, int mode);
        void onError(String error);
        void onProbeInfo(String model, String firmware);
    }

    private ProbeCallback callback;

    // Puertos
    private static final int STREAM_PORT = 5004;
    private static final int CONTROL_PORT = 5005;
    private static final int COMMAND_PORT = 5006;

    // Buffer
    private byte[] frameBuffer;
    private int frameWidth = 640;
    private int frameHeight = 480;
    private boolean isStreaming = false;

    // ===================== METODOS PUBLICOS =====================

    public void setCallback(ProbeCallback callback) {
        this.callback = callback;
        this.mainHandler = new Handler(Looper.getMainLooper());
        this.executor = Executors.newFixedThreadPool(4);
    }

    public void connect(String ipAddress) {
        if (state == ProbeState.CONNECTED || state == ProbeState.STREAMING) {
            disconnect();
        }

        setState(ProbeState.CONNECTING);

        executor.execute(() -> {
            try {
                // 1. TCP Connection
                socket = new Socket();
                socket.setSoTimeout(5000);
                socket.connect(new InetSocketAddress(ipAddress, STREAM_PORT), 3000);

                outputStream = socket.getOutputStream();
                inputStream = socket.getInputStream();

                // 2. Handshake
                performHandshake();

                // 3. Get probe info
                requestProbeInfo();

                // 4. Set state
                setState(ProbeState.CONNECTED);

                // 5. Start frame listener
                startFrameListener();

            } catch (Exception e) {
                setState(ProbeState.ERROR);
                notifyError("Connection failed: " + e.getMessage());
            }
        });
    }

    public void disconnect() {
        isStreaming = false;

        if (inputStream != null) {
            try { inputStream.close(); } catch (Exception e) {}
        }
        if (outputStream != null) {
            try { outputStream.close(); } catch (Exception e) {}
        }
        if (socket != null) {
            try { socket.close(); } catch (Exception e) {}
        }

        setState(ProbeState.DISCONNECTED);
    }

    public void startStreaming() {
        if (state != ProbeState.CONNECTED) return;

        isStreaming = true;
        setState(ProbeState.STREAMING);

        // Send start command
        sendCommand("START_STREAM");
    }

    public void stopStreaming() {
        isStreaming = false;
        sendCommand("STOP_STREAM");
        setState(ProbeState.CONNECTED);
    }

    public void freeze() {
        sendCommand("FREEZE");
    }

    public void unfreeze() {
        sendCommand("UNFREEZE");
    }

    // ===================== CONTROLES DE IMAGEN =====================

    public void setGain(int gain) {
        sendParameter("GAIN", gain);
    }

    public void setDepth(int depth) {
        sendParameter("DEPTH", depth);
    }

    public void setDynamicRange(int dr) {
        sendParameter("DR", dr);
    }

    public void setEnhancement(int enhance) {
        sendParameter("ENHANCE", enhance);
    }

    public void setHarmonic(boolean enabled) {
        sendParameter("HARMONIC", enabled ? 1 : 0);
    }

    public void setFocus(int focus) {
        sendParameter("FOCUS", focus);
    }

    public void setZoom(int zoom) {
        sendParameter("ZOOM", zoom);
    }

    public void setCompounding(boolean enabled) {
        sendParameter("COMPOUND", enabled ? 1 : 0);
    }

    public void setAcousticalPower(int power) {
        sendParameter("POWER", power);
    }

    // ===================== COLOR FLOW =====================

    public void setColorGain(int gain) {
        sendParameter("BC_GAIN", gain);
    }

    public void setWallFilter(int filter) {
        sendParameter("BC_WF", filter);
    }

    public void setColorPRF(int prf) {
        sendParameter("BC_PRF", prf);
    }

    public void setDopplerAngle(int angle) {
        sendParameter("BD_ANGLE", angle);
    }

    public void setDopplerPRF(int prf) {
        sendParameter("BD_PRF", prf);
    }

    // ===================== MODO =====================

    public void setMode(String mode) {
        sendCommand("SET_MODE:" + mode);
    }

    public void captureImage() {
        sendCommand("CAPTURE");
    }

    public void saveImage(String path) {
        sendCommand("SAVE:" + path);
    }

    // ===================== INTERNOS =====================

    private void performHandshake() throws Exception {
        byte[] handshake = "BYTEC_US_PRO\0".getBytes();
        outputStream.write(handshake);
        outputStream.flush();

        // Read response
        byte[] response = new byte[32];
        int read = inputStream.read(response);
        String responseStr = new String(response, 0, read).trim();

        if (!responseStr.startsWith("OK")) {
            throw new Exception("Handshake failed: " + responseStr);
        }
    }

    private void requestProbeInfo() throws Exception {
        byte[] cmd = "GET_INFO\0".getBytes();
        outputStream.write(cmd);
        outputStream.flush();

        byte[] response = new byte[256];
        int read = inputStream.read(response);
        String info = new String(response, 0, read).trim();

        // Parse: MODEL:XXX;FW:XXX;TYPE:XXX
        String[] parts = info.split(";");
        for (String part : parts) {
            String[] kv = part.split(":");
            if (kv.length == 2) {
                switch (kv[0]) {
                    case "MODEL": probeModel = kv[1]; break;
                    case "FW": firmwareVersion = kv[1]; break;
                    case "TYPE":
                        probeType = ProbeType.valueOf(kv[1].toUpperCase());
                        break;
                }
            }
        }

        notifyProbeInfo(probeModel, firmwareVersion);
    }

    private void startFrameListener() {
        executor.execute(() -> {
            while (isStreaming && !socket.isClosed()) {
                try {
                    // Read frame header
                    byte[] header = readBytes(9);
                    int mode = header[0] & 0xFF;
                    int width = ((header[1] & 0xFF) << 8) | (header[2] & 0xFF);
                    int height = ((header[3] & 0xFF) << 8) | (header[4] & 0xFF);
                    int dataSize = ((header[5] & 0xFF) << 24) |
                                   ((header[6] & 0xFF) << 16) |
                                   ((header[7] & 0xFF) << 8) |
                                   (header[8] & 0xFF);

                    // Read frame data
                    byte[] data = readBytes(dataSize);

                    frameWidth = width;
                    frameHeight = height;

                    // Notify
                    notifyFrame(data, width, height, mode);

                } catch (Exception e) {
                    if (isStreaming) {
                        notifyError("Frame read error: " + e.getMessage());
                    }
                }
            }
        });
    }

    private byte[] readBytes(int count) throws Exception {
        byte[] buffer = new byte[count];
        int offset = 0;
        while (offset < count) {
            int read = inputStream.read(buffer, offset, count - offset);
            if (read == -1) throw new EOFException("Stream ended");
            offset += read;
        }
        return buffer;
    }

    private void sendCommand(String command) {
        executor.execute(() -> {
            try {
                if (outputStream != null) {
                    outputStream.write((command + "\0").getBytes());
                    outputStream.flush();
                }
            } catch (Exception e) {
                notifyError("Send command failed: " + e.getMessage());
            }
        });
    }

    private void sendParameter(String param, int value) {
        sendCommand("SET:" + param + "=" + value);
    }

    // ===================== STATE MANAGEMENT =====================

    private void setState(ProbeState newState) {
        state = newState;
        if (callback != null) {
            mainHandler.post(() -> callback.onStateChanged(newState));
        }
    }

    private void notifyFrame(byte[] data, int w, int h, int mode) {
        if (callback != null) {
            mainHandler.post(() -> callback.onFrameReceived(data, w, h, mode));
        }
    }

    private void notifyError(String error) {
        if (callback != null) {
            mainHandler.post(() -> callback.onError(error));
        }
    }

    private void notifyProbeInfo(String model, String fw) {
        if (callback != null) {
            mainHandler.post(() -> callback.onProbeInfo(model, fw));
        }
    }

    // ===================== GETTERS =====================

    public ProbeState getState() { return state; }
    public ConnectionType getConnectionType() { return connectionType; }
    public ProbeType getProbeType() { return probeType; }
    public String getProbeId() { return probeId; }
    public String getProbeModel() { return probeModel; }
    public String getFirmwareVersion() { return firmwareVersion; }
    public int getFrameWidth() { return frameWidth; }
    public int getFrameHeight() { return frameHeight; }
    public boolean isConnected() { return state == ProbeState.CONNECTED || state == ProbeState.STREAMING; }
    public boolean isStreaming() { return state == ProbeState.STREAMING; }
}
```

---

# 8. MODULO: IMAGING

## 8.1 ImageProcessor.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.graphics.*;
import java.nio.ByteBuffer;

public class ImageProcessor {

    // ===================== DSC (Digital Scan Conversion) =====================

    public static Bitmap digitalScanConversion(byte[] rfData, int rfWidth, int rfHeight,
                                                int outputWidth, int outputHeight) {
        Bitmap bitmap = Bitmap.createBitmap(outputWidth, outputHeight, Bitmap.Config.ARGB_8888);
        int[] pixels = new int[outputWidth * outputHeight];

        for (int y = 0; y < outputHeight; y++) {
            for (int x = 0; x < outputWidth; x++) {
                // Convert cartesian to polar
                double angle = Math.atan2(x - outputWidth/2.0, y);
                double depth = Math.sqrt(
                    Math.pow(x - outputWidth/2.0, 2) + Math.pow(y, 2));

                // Map to RF data
                int rfX = (int)(angle * rfWidth / Math.PI) + rfWidth/2;
                int rfY = (int)(depth * rfHeight / outputHeight);

                if (rfX >= 0 && rfX < rfWidth && rfY >= 0 && rfY < rfHeight) {
                    int value = rfData[rfY * rfWidth + rfX] & 0xFF;
                    pixels[y * outputWidth + x] = Color.rgb(value, value, value);
                }
            }
        }

        bitmap.setPixels(pixels, 0, outputWidth, 0, 0, outputWidth, outputHeight);
        return bitmap;
    }

    // ===================== ENHANCEMENT =====================

    public static Bitmap edgeEnhancement(Bitmap input, int level) {
        int width = input.getWidth();
        int height = input.getHeight();
        int[] pixels = new int[width * height];
        input.getPixels(pixels, 0, width, 0, 0, width, height);

        int[] output = new int[pixels.length];

        // Laplacian kernel
        int[] kernel = {
            0, -1, 0,
           -1,  4, -1,
            0, -1, 0
        };

        for (int y = 1; y < height - 1; y++) {
            for (int x = 1; x < width - 1; x++) {
                int sum = 0;
                for (int ky = -1; ky <= 1; ky++) {
                    for (int kx = -1; kx <= 1; kx++) {
                        int idx = (y + ky) * width + (x + kx);
                        int gray = Color.red(pixels[idx]);
                        sum += gray * kernel[(ky + 1) * 3 + (kx + 1)];
                    }
                }

                int idx = y * width + x;
                int gray = Color.red(pixels[idx]);
                int enhanced = Math.min(255, Math.max(0, gray + sum * level / 4));
                output[idx] = Color.rgb(enhanced, enhanced, enhanced);
            }
        }

        Bitmap result = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        result.setPixels(output, 0, width, 0, 0, width, height);
        return result;
    }

    public static Bitmap tissueHarmonicImaging(Bitmap input) {
        int width = input.getWidth();
        int height = input.getHeight();
        int[] pixels = new int[width * height];
        input.getPixels(pixels, 0, width, 0, 0, width, height);

        int[] output = new int[pixels.length];

        for (int i = 0; i < pixels.length; i++) {
            int gray = Color.red(pixels[i]);
            // Simulate harmonic: reduce low-frequency components
            int harmonic = (int)(gray * 0.85 + 20);
            harmonic = Math.min(255, Math.max(0, harmonic));
            output[i] = Color.rgb(harmonic, harmonic, harmonic);
        }

        Bitmap result = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        result.setPixels(output, 0, width, 0, 0, width, height);
        return result;
    }

    public static Bitmap spatialCompounding(Bitmap[] frames) {
        if (frames.length == 0) return null;

        int width = frames[0].getWidth();
        int height = frames[0].getHeight();
        int[] output = new int[width * height];

        // Average multiple frames
        int[][] allPixels = new int[frames.length][];
        for (int f = 0; f < frames.length; f++) {
            allPixels[f] = new int[width * height];
            frames[f].getPixels(allPixels[f], 0, width, 0, 0, width, height);
        }

        for (int i = 0; i < output.length; i++) {
            int sum = 0;
            for (int f = 0; f < frames.length; f++) {
                sum += Color.red(allPixels[f][i]);
            }
            int avg = sum / frames.length;
            output[i] = Color.rgb(avg, avg, avg);
        }

        Bitmap result = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        result.setPixels(output, 0, width, 0, 0, width, height);
        return result;
    }

    // ===================== COLOR DOPPLER =====================

    public static Bitmap colorDopplerMap(byte[] colorData, int width, int height,
                                         int wallFilter, int prf) {
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        int[] pixels = new int[width * height];

        for (int i = 0; i < colorData.length; i += 4) {
            int r = colorData[i] & 0xFF;
            int g = colorData[i + 1] & 0xFF;
            int b = colorData[i + 2] & 0xFF;
            int a = colorData[i + 3] & 0xFF;

            // Apply wall filter
            if (a < wallFilter * 25) {
                pixels[i / 4] = 0; // transparent
            } else {
                pixels[i / 4] = Color.argb(a, r, g, b);
            }
        }

        bitmap.setPixels(pixels, 0, width, 0, 0, width, height);
        return bitmap;
    }

    // ===================== PW SPECTRAL =====================

    public static Bitmap pwSpectralDisplay(float[] spectralData, int width, int height,
                                           float baseline, float scale) {
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);

        // Black background
        canvas.drawColor(Color.BLACK);

        // Draw baseline
        Paint baselinePaint = new Paint();
        baselinePaint.setColor(Color.GRAY);
        baselinePaint.setStrokeWidth(1);
        float baselineY = height * (1 - baseline) / 2;
        canvas.drawLine(0, baselineY, width, baselineY, baselinePaint);

        // Draw spectrum
        Paint spectrumPaint = new Paint();
        spectrumPaint.setColor(Color.GREEN);
        spectrumPaint.setStrokeWidth(2);
        spectrumPaint.setStyle(Paint.Style.STROKE);

        Path path = new Path();
        float pointWidth = (float)width / spectralData.length;

        for (int i = 0; i < spectralData.length; i++) {
            float x = i * pointWidth;
            float y = baselineY - (spectralData[i] * scale * height / 2);

            if (i == 0) {
                path.moveTo(x, y);
            } else {
                path.lineTo(x, y);
            }
        }

        canvas.drawPath(path, spectrumPaint);

        // Fill positive area
        Paint fillPaint = new Paint();
        fillPaint.setColor(Color.argb(50, 0, 230, 118));
        fillPaint.setStyle(Paint.Style.FILL);

        Path fillPath = new Path(path);
        fillPath.lineTo(width, baselineY);
        fillPath.lineTo(0, baselineY);
        fillPath.close();
        canvas.drawPath(fillPath, fillPaint);

        return bitmap;
    }

    // ===================== M-MODE =====================

    public static Bitmap mModeDisplay(int[][] mModeLines, int width, int height) {
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        int[] pixels = new int[width * height];

        float lineWidth = (float)width / mModeLines.length;

        for (int x = 0; x < mModeLines.length && x < width; x++) {
            int[] line = mModeLines[x];
            float pointHeight = (float)height / line.length;

            for (int y = 0; y < line.length && y < height; y++) {
                int gray = line[y];
                int px = (int)(x * lineWidth);
                int py = (int)(y * pointHeight);
                if (py < height && px < width) {
                    pixels[py * width + px] = Color.rgb(gray, gray, gray);
                }
            }
        }

        bitmap.setPixels(pixels, 0, width, 0, 0, width, height);
        return bitmap;
    }
}
```

---

# 9. MODULO: DICOM

## 9.1 DicomService.smali (Referencia Java Completa)

```java
package com.bytec.uspro;

import java.io.*;
import java.net.*;
import java.nio.ByteBuffer;
import java.security.SecureRandom;
import java.util.*;
import javax.net.ssl.*;

public class DicomService {

    // ===================== CONFIGURACION =====================

    private String localAeTitle = "BYTEC_US";
    private String remoteAeTitle = "PACS";
    private String remoteHost = "192.168.1.100";
    private int remotePort = 11112;

    private String mwlAeTitle = "MWL";
    private String mwlHost = "192.168.1.101";
    private int mwlPort = 11113;

    private Socket socket;
    private OutputStream out;
    private InputStream in;

    // DICOM UIDs
    public static final String UID_Verification = "1.2.840.10008.1.1";
    public static final String UID_CTImageStorage = "1.2.840.10008.5.1.4.1.1.2";
    public static final String UID_USImageStorage = "1.2.840.10008.5.1.4.1.1.6.1";
    public static final String UID_MWL = "1.2.840.10008.5.1.4.31.1";
    public static final String UID_ImplicitVRLittleEndian = "1.2.840.10008.1.2";
    public static final String UID_ExplicitVRLittleEndian = "1.2.840.10008.1.2.1";

    // ===================== INICIALIZACION =====================

    public DicomService(String localAe, String remoteAe, String host, int port) {
        this.localAeTitle = localAe;
        this.remoteAeTitle = remoteAe;
        this.remoteHost = host;
        this.remotePort = port;
    }

    public void configureMwl(String aeTitle, String host, int port) {
        this.mwlAeTitle = aeTitle;
        this.mwlHost = host;
        this.mwlPort = port;
    }

    // ===================== C-ECHO =====================

    public boolean echo() {
        try {
            connect(remoteHost, remotePort);
            sendAssociationRequest();
            receiveAssociationAccept();
            sendCEcho();
            receiveResponse();
            sendRelease();
            disconnect();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // ===================== C-STORE =====================

    public boolean storeImage(byte[] pixelData, Map<String, String> patientInfo,
                               Map<String, String> studyInfo) {
        try {
            connect(remoteHost, remotePort);
            sendAssociationRequest();
            receiveAssociationAccept();

            // Create DICOM dataset
            byte[] dataset = createDicomDataset(pixelData, patientInfo, studyInfo);

            // Send C-STORE
            sendCStore(dataset);
            receiveResponse();

            sendRelease();
            disconnect();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // ===================== C-FIND =====================

    public List<Map<String, String>> find(Map<String, String> queryKeys) {
        List<Map<String, String>> results = new ArrayList<>();

        try {
            connect(remoteHost, remotePort);
            sendAssociationRequest();
            receiveAssociationAccept();

            // Send C-FIND
            sendCFind(queryKeys);

            // Receive results
            while (true) {
                Map<String, String> dataset = receiveDataset();
                if (dataset == null) break;
                if (dataset.containsKey("00080005")) { // Retired
                    break;
                }
                results.add(dataset);
            }

            sendRelease();
            disconnect();
        } catch (Exception e) {
            // Handle error
        }

        return results;
    }

    // ===================== C-MOVE =====================

    public boolean move(String studyUid, String destinationAe) {
        try {
            connect(remoteHost, remotePort);
            sendAssociationRequest();
            receiveAssociationAccept();

            // Send C-MOVE
            sendCMove(studyUid, destinationAe);

            // Wait for completion
            while (true) {
                Map<String, String> response = receiveDataset();
                if (response == null) break;
                String status = response.get("00080005");
                if ("0".equals(status)) break; // Success
            }

            sendRelease();
            disconnect();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // ===================== MWL =====================

    public List<Map<String, String>> queryWorklist(String patientName, String studyDate) {
        Map<String, String> queryKeys = new HashMap<>();
        queryKeys.put("00100010", patientName); // PatientName
        queryKeys.put("00080020", studyDate);   // StudyDate
        queryKeys.put("00080060", "US");        // Modality
        queryKeys.put("00400001", mwlAeTitle);  // ScheduledStationAET

        // Connect to MWL server
        String originalHost = remoteHost;
        int originalPort = remotePort;
        String originalAe = remoteAeTitle;

        remoteHost = mwlHost;
        remotePort = mwlPort;
        remoteAeTitle = mwlAeTitle;

        List<Map<String, String>> results = find(queryKeys);

        // Restore original
        remoteHost = originalHost;
        remotePort = originalPort;
        remoteAeTitle = originalAe;

        return results;
    }

    // ===================== MPPS =====================

    public boolean sendMpps(String status) {
        try {
            connect(remoteHost, remotePort);
            sendAssociationRequest();
            receiveAssociationAccept();

            // N-EVENT-REPORT with MPPS
            sendNEventReport(status);

            sendRelease();
            disconnect();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // ===================== DICOM PROTOCOL =====================

    private void connect(String host, int port) throws Exception {
        socket = new Socket(host, port);
        socket.setSoTimeout(10000);
        out = socket.getOutputStream();
        in = socket.getInputStream();
    }

    private void disconnect() {
        try {
            if (socket != null) socket.close();
        } catch (Exception e) {}
    }

    private void sendAssociationRequest() throws Exception {
        // A-ASSOCIATE-RQ
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        // PDU Type: A-ASSOCIATE-RQ (0x01)
        buffer.put((byte)0x01);

        // Reserved
        buffer.put((byte)0x00);

        // PDU Length (placeholder)
        int pduLength = 0;
        int pduLengthPos = buffer.position();
        buffer.putInt(0);

        // Protocol Version
        buffer.putShort((short)0x0001);

        // Reserved
        buffer.putShort((short)0x0000);

        // Called AE Title
        byte[] calledAe = new byte[16];
        byte[] calledBytes = remoteAeTitle.getBytes();
        System.arraycopy(calledBytes, 0, calledAe, 0, Math.min(calledBytes.length, 16));
        buffer.put(calledAe);

        // Calling AE Title
        byte[] callingAe = new byte[16];
        byte[] callingBytes = localAeTitle.getBytes();
        System.arraycopy(callingBytes, 0, callingAe, 0, Math.min(callingBytes.length, 16));
        buffer.put(callingAe);

        // Reserved bytes (32)
        buffer.put(new byte[32]);

        // Application Context
        buffer.put((byte)0x10); // Application Context Item
        buffer.put((byte)0x00);
        String appContext = "1.2.840.10008.3.1.1.1";
        byte[] appContextBytes = appContext.getBytes();
        buffer.putShort((short)appContextBytes.length);
        buffer.put(appContextBytes);

        // Presentation Context
        buffer.put((byte)0x20); // Presentation Context Item
        buffer.put((byte)0x00);

        // Abstract Syntax (US Image Storage)
        String abstractSyntax = UID_USImageStorage;
        byte[] abstractBytes = abstractSyntax.getBytes();

        // Transfer Syntax
        String transferSyntax = UID_ExplicitVRLittleEndian;
        byte[] transferBytes = transferSyntax.getBytes();

        int contextLength = 4 + abstractBytes.length + 4 + transferBytes.length;
        buffer.putShort((short)contextLength);

        // Abstract Syntax Item
        buffer.put((byte)0x30);
        buffer.put((byte)0x00);
        buffer.putShort((short)abstractBytes.length);
        buffer.put(abstractBytes);

        // Transfer Syntax Item
        buffer.put((byte)0x40);
        buffer.put((byte)0x00);
        buffer.putShort((short)transferBytes.length);
        buffer.put(transferBytes);

        // User Information
        buffer.put((byte)0x50); // User Information Item
        buffer.put((byte)0x00);
        byte[] userInfo = new byte[0]; // TODO: Add max PDU length
        buffer.putShort((short)userInfo.length);
        buffer.put(userInfo);

        // Update PDU length
        pduLength = buffer.position() - 6;
        buffer.putInt(pduLengthPos, pduLength);

        // Send
        byte[] data = new byte[buffer.position()];
        buffer.flip();
        buffer.get(data);
        out.write(data);
        out.flush();
    }

    private void receiveAssociationAccept() throws Exception {
        byte[] header = readBytes(6);
        int pduType = header[0] & 0xFF;

        if (pduType != 0x02) { // A-ASSOCIATE-AC
            throw new Exception("Expected A-ASSOCIATE-AC, got: " + pduType);
        }

        int pduLength = ((header[2] & 0xFF) << 24) |
                        ((header[3] & 0xFF) << 16) |
                        ((header[4] & 0xFF) << 8) |
                        (header[5] & 0xFF);

        readBytes(pduLength); // Read rest of PDU
    }

    private void sendCEcho() throws Exception {
        ByteBuffer buffer = ByteBuffer.allocate(256);

        // PDU Type: P-DATA (0x04)
        buffer.put((byte)0x04);
        buffer.put((byte)0x00);

        // PDU Length placeholder
        int pduLengthPos = buffer.position();
        buffer.putInt(0);

        // Presentation Context ID
        buffer.put((byte)0x01);
        buffer.put((byte)0x00);

        // Command Set
        buffer.putShort((byte)0x00, (byte)0x01); // Command Group Length
        buffer.putShort((byte)0x00, (byte)0x02);
        buffer.putInt(0); // Length placeholder

        buffer.putShort((byte)0x00, (byte)0x03); // Command Field
        buffer.putShort((byte)0x00, (byte)0x02);
        buffer.putShort((short)0x0030); // C-ECHO-RQ

        buffer.putShort((byte)0x00, (byte)0x01); // Message ID
        buffer.putShort((byte)0x00, (byte)0x02);
        buffer.putInt(1);

        buffer.putShort((byte)0x00, (byte)0x02); // Affected SOP Class UID
        buffer.putShort((byte)0x00, (byte)0x00);
        String sopClass = UID_Verification;
        buffer.putShort((short)sopClass.getBytes().length);
        buffer.put(sopClass.getBytes());

        buffer.putShort((byte)0x00, (byte)0x03); // Affected SOP Instance UID
        buffer.putShort((byte)0x00, (byte)0x00);
        String sopInstance = "1.2.3.4.5";
        buffer.putShort((short)sopInstance.getBytes().length);
        buffer.put(sopInstance.getBytes());

        // Update lengths
        buffer.flip();
        byte[] data = new byte[buffer.remaining()];
        buffer.get(data);
        out.write(data);
        out.flush();
    }

    private void sendCStore(byte[] dataset) throws Exception {
        // Similar to C-ECHO but with dataset
        // TODO: Implement full C-STORE
    }

    private void sendCFind(Map<String, String> keys) throws Exception {
        // TODO: Implement C-FIND
    }

    private void sendCMove(String studyUid, String destinationAe) throws Exception {
        // TODO: Implement C-MOVE
    }

    private void sendNEventReport(String status) throws Exception {
        // TODO: Implement N-EVENT-REPORT (MPPS)
    }

    private void sendRelease() throws Exception {
        ByteBuffer buffer = ByteBuffer.allocate(6);
        buffer.put((byte)0x05); // A-RELEASE-RQ
        buffer.put((byte)0x00);
        buffer.putInt(0);
        out.write(buffer.array());
        out.flush();
    }

    private void receiveResponse() throws Exception {
        readBytes(6); // Read response header
    }

    private Map<String, String> receiveDataset() throws Exception {
        // TODO: Parse DICOM dataset from stream
        return null;
    }

    private byte[] createDicomDataset(byte[] pixelData,
                                       Map<String, String> patientInfo,
                                       Map<String, String> studyInfo) {
        // TODO: Create proper DICOM dataset
        return new byte[0];
    }

    private byte[] readBytes(int count) throws Exception {
        byte[] buffer = new byte[count];
        int offset = 0;
        while (offset < count) {
            int read = in.read(buffer, offset, count - offset);
            if (read == -1) throw new EOFException();
            offset += read;
        }
        return buffer;
    }

    // ===================== GENERADORES =====================

    private String generateUid() {
        SecureRandom random = new SecureRandom();
        StringBuilder uid = new StringBuilder("1.2.826.0.1.3680043.");
        uid.append(System.currentTimeMillis());
        uid.append(".");
        uid.append(random.nextInt(10000));
        return uid.toString();
    }

    private String formatDate(java.util.Date date) {
        java.text.SimpleDateFormat sdf = new java.text.SimpleDateFormat("yyyyMMdd");
        return sdf.format(date);
    }

    private String formatTime(java.util.Date date) {
        java.text.SimpleDateFormat sdf = new java.text.SimpleDateFormat("HHmmss");
        return sdf.format(date);
    }
}
```

---

# 10. MODULO: PATIENT DB

## 10.1 PatientDatabase.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.content.*;
import android.database.*;
import android.database.sqlite.*;
import java.util.*;

public class PatientDatabase {

    private static final String DB_NAME = "bytec_patients.db";
    private static final int DB_VERSION = 1;
    private SQLiteDatabase db;
    private Context context;

    // Singleton
    private static PatientDatabase instance;

    public static synchronized PatientDatabase getInstance(Context context) {
        if (instance == null) {
            instance = new PatientDatabase(context);
        }
        return instance;
    }

    private PatientDatabase(Context context) {
        this.context = context.getApplicationContext();
        db = new DatabaseHelper(this.context).getWritableDatabase();
    }

    // ===================== TABLE CREATION =====================

    private static class DatabaseHelper extends SQLiteOpenHelper {

        DatabaseHelper(Context context) {
            super(context, DB_NAME, null, DB_VERSION);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            // Patients table
            db.execSQL("CREATE TABLE patients (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "dicom_id TEXT," +
                "name TEXT NOT NULL," +
                "birth_date TEXT," +
                "sex TEXT," +
                "age INTEGER," +
                "phone TEXT," +
                "email TEXT," +
                "address TEXT," +
                "notes TEXT," +
                "referring_physician TEXT," +
                "created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP," +
                "updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP," +
                "is_deleted INTEGER DEFAULT 0" +
            ")");

            // Studies table
            db.execSQL("CREATE TABLE studies (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "patient_id INTEGER NOT NULL," +
                "study_uid TEXT UNIQUE," +
                "study_date TEXT," +
                "study_time TEXT," +
                "description TEXT," +
                "modality TEXT DEFAULT 'US'," +
                "accession_number TEXT," +
                "institution_name TEXT DEFAULT 'ByteC Medical'," +
                "images_count INTEGER DEFAULT 0," +
                "dicom_sent INTEGER DEFAULT 0," +
                "dicom_sent_at TIMESTAMP," +
                "created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP," +
                "is_deleted INTEGER DEFAULT 0," +
                "FOREIGN KEY(patient_id) REFERENCES patients(id)" +
            ")");

            // Images table
            db.execSQL("CREATE TABLE images (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "study_id INTEGER NOT NULL," +
                "instance_uid TEXT UNIQUE," +
                "file_path TEXT NOT NULL," +
                "thumbnail_path TEXT," +
                "image_type TEXT," +
                "width INTEGER," +
                "height INTEGER," +
                "file_size INTEGER," +
                "capture_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP," +
                "probe_model TEXT," +
                "gain INTEGER," +
                "depth INTEGER," +
                "dynamic_range INTEGER," +
                "harmonic INTEGER," +
                "focus_position INTEGER," +
                "dicom_sent INTEGER DEFAULT 0," +
                "annotations TEXT," +
                "measurements TEXT," +
                "is_deleted INTEGER DEFAULT 0," +
                "FOREIGN KEY(study_id) REFERENCES studies(id)" +
            ")");

            // Measurements table
            db.execSQL("CREATE TABLE measurements (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                "image_id INTEGER NOT NULL," +
                "measurement_type TEXT NOT NULL," +
                "value REAL," +
                "unit TEXT," +
                "points TEXT," +
                "label TEXT," +
                "created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP," +
                "FOREIGN KEY(image_id) REFERENCES images(id)" +
            ")");

            // Settings table
            db.execSQL("CREATE TABLE settings (" +
                "key TEXT PRIMARY KEY," +
                "value TEXT," +
                "updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP" +
            ")");

            // Create indexes
            db.execSQL("CREATE INDEX idx_patients_name ON patients(name)");
            db.execSQL("CREATE INDEX idx_studies_patient ON studies(patient_id)");
            db.execSQL("CREATE INDEX idx_images_study ON images(study_id)");
            db.execSQL("CREATE INDEX idx_images_capture ON images(capture_time)");
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            // TODO: Handle upgrades
        }
    }

    // ===================== PATIENT OPERATIONS =====================

    public long insertPatient(Patient patient) {
        ContentValues values = new ContentValues();
        values.put("dicom_id", patient.dicomId);
        values.put("name", patient.name);
        values.put("birth_date", patient.birthDate);
        values.put("sex", patient.sex);
        values.put("age", patient.age);
        values.put("phone", patient.phone);
        values.put("email", patient.email);
        values.put("address", patient.address);
        values.put("notes", patient.notes);
        values.put("referring_physician", patient.referringPhysician);

        return db.insert("patients", null, values);
    }

    public int updatePatient(Patient patient) {
        ContentValues values = new ContentValues();
        values.put("name", patient.name);
        values.put("birth_date", patient.birthDate);
        values.put("sex", patient.sex);
        values.put("phone", patient.phone);
        values.put("email", patient.email);
        values.put("notes", patient.notes);
        values.put("referring_physician", patient.referringPhysician);
        values.put("updated_at", System.currentTimeMillis());

        return db.update("patients", values, "id=?",
            new String[]{String.valueOf(patient.id)});
    }

    public void deletePatient(long id) {
        // Soft delete
        ContentValues values = new ContentValues();
        values.put("is_deleted", 1);
        db.update("patients", values, "id=?", new String[]{String.valueOf(id)});
    }

    public Patient getPatient(long id) {
        Cursor cursor = db.query("patients", null, "id=? AND is_deleted=0",
            new String[]{String.valueOf(id)}, null, null, null);

        Patient patient = null;
        if (cursor.moveToFirst()) {
            patient = cursorToPatient(cursor);
        }
        cursor.close();
        return patient;
    }

    public List<Patient> searchPatients(String query) {
        List<Patient> patients = new ArrayList<>();

        Cursor cursor = db.query("patients", null,
            "(name LIKE ? OR dicom_id LIKE ?) AND is_deleted=0",
            new String[]{"%" + query + "%", "%" + query + "%"},
            null, null, "name ASC", "50");

        while (cursor.moveToNext()) {
            patients.add(cursorToPatient(cursor));
        }
        cursor.close();
        return patients;
    }

    public List<Patient> getAllPatients() {
        List<Patient> patients = new ArrayList<>();

        Cursor cursor = db.query("patients", null, "is_deleted=0",
            null, null, null, "name ASC");

        while (cursor.moveToNext()) {
            patients.add(cursorToPatient(cursor));
        }
        cursor.close();
        return patients;
    }

    private Patient cursorToPatient(Cursor cursor) {
        Patient p = new Patient();
        p.id = cursor.getLong(cursor.getColumnIndexOrThrow("id"));
        p.dicomId = cursor.getString(cursor.getColumnIndexOrThrow("dicom_id"));
        p.name = cursor.getString(cursor.getColumnIndexOrThrow("name"));
        p.birthDate = cursor.getString(cursor.getColumnIndexOrThrow("birth_date"));
        p.sex = cursor.getString(cursor.getColumnIndexOrThrow("sex"));
        p.age = cursor.getInt(cursor.getColumnIndexOrThrow("age"));
        p.phone = cursor.getString(cursor.getColumnIndexOrThrow("phone"));
        p.email = cursor.getString(cursor.getColumnIndexOrThrow("email"));
        p.address = cursor.getString(cursor.getColumnIndexOrThrow("address"));
        p.notes = cursor.getString(cursor.getColumnIndexOrThrow("notes"));
        p.referringPhysician = cursor.getString(cursor.getColumnIndexOrThrow("referring_physician"));
        return p;
    }

    // ===================== STUDY OPERATIONS =====================

    public long insertStudy(Study study) {
        ContentValues values = new ContentValues();
        values.put("patient_id", study.patientId);
        values.put("study_uid", study.studyUid);
        values.put("study_date", study.studyDate);
        values.put("study_time", study.studyTime);
        values.put("description", study.description);
        values.put("modality", study.modality);
        values.put("accession_number", study.accessionNumber);

        return db.insert("studies", null, values);
    }

    public List<Study> getPatientStudies(long patientId) {
        List<Study> studies = new ArrayList<>();

        Cursor cursor = db.query("studies", null,
            "patient_id=? AND is_deleted=0",
            new String[]{String.valueOf(patientId)},
            null, null, "study_date DESC");

        while (cursor.moveToNext()) {
            studies.add(cursorToStudy(cursor));
        }
        cursor.close();
        return studies;
    }

    private Study cursorToStudy(Cursor cursor) {
        Study s = new Study();
        s.id = cursor.getLong(cursor.getColumnIndexOrThrow("id"));
        s.patientId = cursor.getLong(cursor.getColumnIndexOrThrow("patient_id"));
        s.studyUid = cursor.getString(cursor.getColumnIndexOrThrow("study_uid"));
        s.studyDate = cursor.getString(cursor.getColumnIndexOrThrow("study_date"));
        s.studyTime = cursor.getString(cursor.getColumnIndexOrThrow("study_time"));
        s.description = cursor.getString(cursor.getColumnIndexOrThrow("description"));
        s.modality = cursor.getString(cursor.getColumnIndexOrThrow("modality"));
        s.accessionNumber = cursor.getString(cursor.getColumnIndexOrThrow("accession_number"));
        s.imagesCount = cursor.getInt(cursor.getColumnIndexOrThrow("images_count"));
        s.dicomSent = cursor.getInt(cursor.getColumnIndexOrThrow("dicom_sent")) == 1;
        return s;
    }

    // ===================== IMAGE OPERATIONS =====================

    public long insertImage(UltrasoundImage image) {
        ContentValues values = new ContentValues();
        values.put("study_id", image.studyId);
        values.put("instance_uid", image.instanceUid);
        values.put("file_path", image.filePath);
        values.put("thumbnail_path", image.thumbnailPath);
        values.put("image_type", image.imageType);
        values.put("width", image.width);
        values.put("height", image.height);
        values.put("file_size", image.fileSize);
        values.put("probe_model", image.probeModel);
        values.put("gain", image.gain);
        values.put("depth", image.depth);
        values.put("dynamic_range", image.dynamicRange);
        values.put("harmonic", image.harmonic ? 1 : 0);
        values.put("focus_position", image.focusPosition);
        values.put("annotations", image.annotationsJson);
        values.put("measurements", image.measurementsJson);

        return db.insert("images", null, values);
    }

    public List<UltrasoundImage> getStudyImages(long studyId) {
        List<UltrasoundImage> images = new ArrayList<>();

        Cursor cursor = db.query("images", null,
            "study_id=? AND is_deleted=0",
            new String[]{String.valueOf(studyId)},
            null, null, "capture_time ASC");

        while (cursor.moveToNext()) {
            images.add(cursorToImage(cursor));
        }
        cursor.close();
        return images;
    }

    private UltrasoundImage cursorToImage(Cursor cursor) {
        UltrasoundImage img = new UltrasoundImage();
        img.id = cursor.getLong(cursor.getColumnIndexOrThrow("id"));
        img.studyId = cursor.getLong(cursor.getColumnIndexOrThrow("study_id"));
        img.instanceUid = cursor.getString(cursor.getColumnIndexOrThrow("instance_uid"));
        img.filePath = cursor.getString(cursor.getColumnIndexOrThrow("file_path"));
        img.thumbnailPath = cursor.getString(cursor.getColumnIndexOrThrow("thumbnail_path"));
        img.imageType = cursor.getString(cursor.getColumnIndexOrThrow("image_type"));
        img.width = cursor.getInt(cursor.getColumnIndexOrThrow("width"));
        img.height = cursor.getInt(cursor.getColumnIndexOrThrow("height"));
        img.fileSize = cursor.getInt(cursor.getColumnIndexOrThrow("file_size"));
        img.probeModel = cursor.getString(cursor.getColumnIndexOrThrow("probe_model"));
        img.gain = cursor.getInt(cursor.getColumnIndexOrThrow("gain"));
        img.depth = cursor.getInt(cursor.getColumnIndexOrThrow("depth"));
        img.dynamicRange = cursor.getInt(cursor.getColumnIndexOrThrow("dynamic_range"));
        img.harmonic = cursor.getInt(cursor.getColumnIndexOrThrow("harmonic")) == 1;
        img.focusPosition = cursor.getInt(cursor.getColumnIndexOrThrow("focus_position"));
        img.annotationsJson = cursor.getString(cursor.getColumnIndexOrThrow("annotations"));
        img.measurementsJson = cursor.getString(cursor.getColumnIndexOrThrow("measurements"));
        return img;
    }

    // ===================== SETTINGS =====================

    public void setSetting(String key, String value) {
        ContentValues values = new ContentValues();
        values.put("key", key);
        values.put("value", value);
        values.put("updated_at", System.currentTimeMillis());
        db.insertWithOnConflict("settings", null, values,
            SQLiteDatabase.CONFLICT_REPLACE);
    }

    public String getSetting(String key, String defaultValue) {
        Cursor cursor = db.query("settings", null, "key=?",
            new String[]{key}, null, null, null);

        String value = defaultValue;
        if (cursor.moveToFirst()) {
            value = cursor.getString(cursor.getColumnIndexOrThrow("value"));
        }
        cursor.close();
        return value;
    }

    // ===================== MODELS =====================

    public static class Patient {
        public long id;
        public String dicomId;
        public String name;
        public String birthDate;
        public String sex;
        public int age;
        public String phone;
        public String email;
        public String address;
        public String notes;
        public String referringPhysician;
    }

    public static class Study {
        public long id;
        public long patientId;
        public String studyUid;
        public String studyDate;
        public String studyTime;
        public String description;
        public String modality;
        public String accessionNumber;
        public String institutionName;
        public int imagesCount;
        public boolean dicomSent;
    }

    public static class UltrasoundImage {
        public long id;
        public long studyId;
        public String instanceUid;
        public String filePath;
        public String thumbnailPath;
        public String imageType;
        public int width;
        public int height;
        public int fileSize;
        public long captureTime;
        public String probeModel;
        public int gain;
        public int depth;
        public int dynamicRange;
        public boolean harmonic;
        public int focusPosition;
        public boolean dicomSent;
        public String annotationsJson;
        public String measurementsJson;
    }
}
```

---

# 11. MODULO: MEDIDAS

## 11.1 MeasurementManager.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.graphics.*;
import java.util.*;

public class MeasurementManager {

    public enum MeasurementType {
        DISTANCE("Distance", "mm"),
        AREA("Area", "cm²"),
        VOLUME("Volume", "cm³"),
        ANGLE("Angle", "°"),
        TRACE("Trace", "mm"),
        VELOCITY("Velocity", "cm/s"),
        HEART_RATE("Heart Rate", "bpm"),
        TI("Thermal Index", ""),
        MI("Mechanical Index", "");

        public final String name;
        public final String unit;

        MeasurementType(String name, String unit) {
            this.name = name;
            this.unit = unit;
        }
    }

    public static class Measurement {
        public String id;
        public MeasurementType type;
        public List<PointF> points;
        public double value;
        public String unit;
        public String label;
        public long timestamp;
        public int color;

        public Measurement() {
            this.id = UUID.randomUUID().toString();
            this.points = new ArrayList<>();
            this.timestamp = System.currentTimeMillis();
        }
    }

    private List<Measurement> measurements = new ArrayList<>();
    private Measurement currentMeasurement;
    private boolean isMeasuring = false;
    private int currentColor = Color.RED;

    private float pixelToMm = 1.0f; // Calibration factor
    private int frameHeight = 480;
    private int depthMm = 150; // Depth in mm

    // Callbacks
    public interface MeasurementCallback {
        void onMeasurementAdded(Measurement measurement);
        void onMeasurementUpdated(Measurement measurement);
        void onMeasurementRemoved(String id);
        void onAllMeasurementsCleared();
    }

    private MeasurementCallback callback;

    public void setCallback(MeasurementCallback callback) {
        this.callback = callback;
    }

    public void setCalibration(int frameHeight, int depthMm) {
        this.frameHeight = frameHeight;
        this.depthMm = depthMm;
        this.pixelToMm = (float)depthMm / frameHeight;
    }

    // ===================== START/STOP =====================

    public void startMeasurement(MeasurementType type) {
        currentMeasurement = new Measurement();
        currentMeasurement.type = type;
        currentMeasurement.unit = type.unit;
        currentMeasurement.color = currentColor;
        isMeasuring = true;
    }

    public void addPoint(float x, float y) {
        if (!isMeasuring || currentMeasurement == null) return;

        currentMeasurement.points.add(new PointF(x, y));
        calculateValue();

        if (callback != null) {
            callback.onMeasurementUpdated(currentMeasurement);
        }
    }

    public Measurement finishMeasurement() {
        if (currentMeasurement == null) return null;

        Measurement m = currentMeasurement;
        measurements.add(m);
        isMeasuring = false;
        currentMeasurement = null;

        if (callback != null) {
            callback.onMeasurementAdded(m);
        }

        return m;
    }

    public void cancelMeasurement() {
        isMeasuring = false;
        currentMeasurement = null;
    }

    // ===================== CALCULATIONS =====================

    private void calculateValue() {
        if (currentMeasurement == null || currentMeasurement.points.isEmpty()) return;

        switch (currentMeasurement.type) {
            case DISTANCE:
                calculateDistance();
                break;
            case AREA:
                calculateArea();
                break;
            case VOLUME:
                calculateVolume();
                break;
            case ANGLE:
                calculateAngle();
                break;
            case TRACE:
                calculateTrace();
                break;
            case VELOCITY:
                calculateVelocity();
                break;
            case HEART_RATE:
                calculateHeartRate();
                break;
        }
    }

    private void calculateDistance() {
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 2) return;

        PointF p1 = points.get(points.size() - 2);
        PointF p2 = points.get(points.size() - 1);

        float dx = p2.x - p1.x;
        float dy = p2.y - p1.y;
        float distancePixels = (float)Math.sqrt(dx * dx + dy * dy);

        currentMeasurement.value = distancePixels * pixelToMm;
    }

    private void calculateArea() {
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 3) return;

        // Shoelace formula
        double area = 0;
        for (int i = 0; i < points.size(); i++) {
            int j = (i + 1) % points.size();
            area += points.get(i).x * points.get(j).y;
            area -= points.get(j).x * points.get(i).y;
        }
        area = Math.abs(area) / 2;

        // Convert to mm² then cm²
        double areaMm2 = area * pixelToMm * pixelToMm;
        currentMeasurement.value = areaMm2 / 100; // cm²
    }

    private void calculateVolume() {
        // Simplified ellipsoid volume: V = (π/6) × L × W × H
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 6) return; // Need 3 dimensions

        // Get bounding box
        float minX = Float.MAX_VALUE, maxX = Float.MIN_VALUE;
        float minY = Float.MAX_VALUE, maxY = Float.MIN_VALUE;

        for (PointF p : points) {
            minX = Math.min(minX, p.x);
            maxX = Math.max(maxX, p.x);
            minY = Math.min(minY, p.y);
            maxY = Math.max(maxY, p.y);
        }

        double l = (maxX - minX) * pixelToMm;
        double w = (maxY - minY) * pixelToMm;
        double h = l * 0.6; // Estimate depth

        currentMeasurement.value = (Math.PI / 6) * l * w * h / 1000; // cm³
    }

    private void calculateAngle() {
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 3) return;

        PointF p1 = points.get(0);
        PointF vertex = points.get(1);
        PointF p2 = points.get(2);

        double v1x = p1.x - vertex.x;
        double v1y = p1.y - vertex.y;
        double v2x = p2.x - vertex.x;
        double v2y = p2.y - vertex.y;

        double dot = v1x * v2x + v1y * v2y;
        double det = v1x * v2y - v1y * v2x;

        currentMeasurement.value = Math.abs(Math.atan2(det, dot) * 180 / Math.PI);
    }

    private void calculateTrace() {
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 2) return;

        double perimeter = 0;
        for (int i = 0; i < points.size(); i++) {
            int j = (i + 1) % points.size();
            PointF p1 = points.get(i);
            PointF p2 = points.get(j);
            double dx = p2.x - p1.x;
            double dy = p2.y - p1.y;
            perimeter += Math.sqrt(dx * dx + dy * dy);
        }

        currentMeasurement.value = perimeter * pixelToMm;
    }

    private void calculateVelocity() {
        // PW Doppler velocity calculation
        // V = (PRF × c) / (2 × fo × cos(θ))
        // c = speed of sound (1540 m/s)
        // fo = transmitted frequency
        // θ = Doppler angle

        List<PointF> points = currentMeasurement.points;
        if (points.size() < 2) return;

        PointF p1 = points.get(0);
        PointF p2 = points.get(1);

        // Calculate spectral shift
        float shift = Math.abs(p2.y - p1.y);

        // Convert to velocity (simplified)
        double velocity = shift * 0.1; // cm/s

        currentMeasurement.value = velocity;
    }

    private void calculateHeartRate() {
        // From M-mode or PW trace
        List<PointF> points = currentMeasurement.points;
        if (points.size() < 2) return;

        // Count peaks in time window
        int peaks = 0;
        for (int i = 1; i < points.size() - 1; i++) {
            PointF prev = points.get(i - 1);
            PointF curr = points.get(i);
            PointF next = points.get(i + 1);

            if (curr.y < prev.y && curr.y < next.y) {
                peaks++;
            }
        }

        // Calculate time window in seconds
        float timeWindow = (points.get(points.size() - 1).x - points.get(0).x) / 30; // 30 fps

        if (timeWindow > 0) {
            currentMeasurement.value = (peaks / timeWindow) * 60; // bpm
        }
    }

    // ===================== DRAWING =====================

    public void drawMeasurements(Canvas canvas) {
        for (Measurement m : measurements) {
            drawMeasurement(canvas, m);
        }

        if (currentMeasurement != null) {
            drawMeasurement(canvas, currentMeasurement);
        }
    }

    private void drawMeasurement(Canvas canvas, Measurement m) {
        Paint linePaint = new Paint();
        linePaint.setColor(m.color);
        linePaint.setStrokeWidth(3);
        linePaint.setStyle(Paint.Style.STROKE);
        linePaint.setAntiAlias(true);

        Paint pointPaint = new Paint();
        pointPaint.setColor(m.color);
        pointPaint.setStyle(Paint.Style.FILL);
        pointPaint.setAntiAlias(true);

        Paint textPaint = new Paint();
        textPaint.setColor(m.color);
        textPaint.setTextSize(24);
        textPaint.setAntiAlias(true);

        // Draw points and lines
        if (m.points.size() > 1) {
            for (int i = 0; i < m.points.size() - 1; i++) {
                PointF p1 = m.points.get(i);
                PointF p2 = m.points.get(i + 1);
                canvas.drawLine(p1.x, p1.y, p2.x, p2.y, linePaint);
            }
        }

        // Draw points
        for (PointF p : m.points) {
            canvas.drawCircle(p.x, p.y, 6, pointPaint);
        }

        // Draw value
        if (!m.points.isEmpty() && m.value > 0) {
            PointF lastPoint = m.points.get(m.points.size() - 1);
            String text = String.format("%.1f %s", m.value, m.unit);
            canvas.drawText(text, lastPoint.x + 10, lastPoint.y - 10, textPaint);
        }
    }

    // ===================== MANAGEMENT =====================

    public void removeMeasurement(String id) {
        Iterator<Measurement> it = measurements.iterator();
        while (it.hasNext()) {
            Measurement m = it.next();
            if (m.id.equals(id)) {
                it.remove();
                if (callback != null) {
                    callback.onMeasurementRemoved(id);
                }
                break;
            }
        }
    }

    public void clearAllMeasurements() {
        measurements.clear();
        if (callback != null) {
            callback.onAllMeasurementsCleared();
        }
    }

    public List<Measurement> getMeasurements() {
        return new ArrayList<>(measurements);
    }

    public Measurement getCurrentMeasurement() {
        return currentMeasurement;
    }

    public boolean isMeasuring() {
        return isMeasuring;
    }

    public void setColor(int color) {
        this.currentColor = color;
    }

    // ===================== EXPORT =====================

    public String exportToJson() {
        StringBuilder json = new StringBuilder("[");
        for (int i = 0; i < measurements.size(); i++) {
            Measurement m = measurements.get(i);
            if (i > 0) json.append(",");
            json.append("{");
            json.append("\"id\":\"").append(m.id).append("\",");
            json.append("\"type\":\"").append(m.type.name()).append("\",");
            json.append("\"value\":").append(m.value).append(",");
            json.append("\"unit\":\"").append(m.unit).append("\",");
            json.append("\"label\":\"").append(m.label != null ? m.label : "").append("\"");
            json.append("}");
        }
        json.append("]");
        return json.toString();
    }
}
```

---

# 12. MODULO: REPORTES

## 12.1 ReportGenerator.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.content.Context;
import android.graphics.*;
import java.io.*;
import java.text.SimpleDateFormat;
import java.util.*;

public class ReportGenerator {

    public static class ReportData {
        public String patientName;
        public String patientId;
        public String patientBirthDate;
        public String patientSex;
        public String referringPhysician;

        public String studyDate;
        public String studyTime;
        public String studyDescription;
        public String accessionNumber;

        public String institutionName = "ByteC Medical";
        public String modality = "US";
        public String manufacturer = "ByteC";
        public String modelName = "US Pro";

        public List<byte[]> images;
        public String observations;
        public String measurementsJson;
    }

    // ===================== HTML REPORT =====================

    public static String generateHtmlReport(ReportData data) {
        StringBuilder html = new StringBuilder();

        html.append("<!DOCTYPE html>\n");
        html.append("<html>\n<head>\n");
        html.append("<meta charset='utf-8'>\n");
        html.append("<title>Ultrasound Report</title>\n");
        html.append("<style>\n");
        html.append(getCssStyles());
        html.append("</style>\n");
        html.append("</head>\n<body>\n");

        // Header
        html.append("<div class='header'>\n");
        html.append("<div class='logo'>ByteC</div>\n");
        html.append("<h1>Ultrasound Report</h1>\n");
        html.append("</div>\n");

        // Patient Info
        html.append("<div class='section'>\n");
        html.append("<h2>Patient Information</h2>\n");
        html.append("<table class='info-table'>\n");
        html.append("<tr><td>Name:</td><td>").append(data.patientName).append("</td></tr>\n");
        html.append("<tr><td>ID:</td><td>").append(data.patientId).append("</td></tr>\n");
        html.append("<tr><td>Birth Date:</td><td>").append(data.patientBirthDate).append("</td></tr>\n");
        html.append("<tr><td>Sex:</td><td>").append(data.patientSex).append("</td></tr>\n");
        html.append("<tr><td>Referring Physician:</td><td>").append(data.referringPhysician).append("</td></tr>\n");
        html.append("</table>\n");
        html.append("</div>\n");

        // Study Info
        html.append("<div class='section'>\n");
        html.append("<h2>Study Information</h2>\n");
        html.append("<table class='info-table'>\n");
        html.append("<tr><td>Date:</td><td>").append(data.studyDate).append("</td></tr>\n");
        html.append("<tr><td>Time:</td><td>").append(data.studyTime).append("</td></tr>\n");
        html.append("<tr><td>Description:</td><td>").append(data.studyDescription).append("</td></tr>\n");
        html.append("<tr><td>Accession #:</td><td>").append(data.accessionNumber).append("</td></tr>\n");
        html.append("<tr><td>Modality:</td><td>").append(data.modality).append("</td></tr>\n");
        html.append("<tr><td>Institution:</td><td>").append(data.institutionName).append("</td></tr>\n");
        html.append("</table>\n");
        html.append("</div>\n");

        // Images
        html.append("<div class='section'>\n");
        html.append("<h2>Images</h2>\n");
        if (data.images != null) {
            for (int i = 0; i < data.images.size(); i++) {
                html.append("<div class='image-container'>\n");
                html.append("<img src='data:image/png;base64,")
                    .append(android.util.Base64.encodeToString(data.images.get(i), android.util.Base64.NO_WRAP))
                    .append("' class='report-image'/>\n");
                html.append("</div>\n");
            }
        }
        html.append("</div>\n");

        // Observations
        html.append("<div class='section'>\n");
        html.append("<h2>Observations</h2>\n");
        html.append("<div class='observations'>").append(data.observations != null ? data.observations : "").append("</div>\n");
        html.append("</div>\n");

        // Footer
        html.append("<div class='footer'>\n");
        html.append("<p>© 2024 ByteC Medical Devices</p>\n");
        html.append("<p>Generated: ").append(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())).append("</p>\n");
        html.append("</div>\n");

        html.append("</body>\n</html>");

        return html.toString();
    }

    private static String getCssStyles() {
        return
            "body { font-family: Arial, sans-serif; margin: 20px; background: #fff; }\n" +
            ".header { text-align: center; border-bottom: 2px solid #1E88E5; padding-bottom: 20px; margin-bottom: 20px; }\n" +
            ".logo { font-size: 24px; font-weight: bold; color: #1E88E5; }\n" +
            ".section { margin-bottom: 20px; }\n" +
            ".section h2 { color: #1E88E5; border-bottom: 1px solid #ddd; padding-bottom: 5px; }\n" +
            ".info-table { width: 100%; border-collapse: collapse; }\n" +
            ".info-table td { padding: 8px; border: 1px solid #ddd; }\n" +
            ".info-table td:first-child { font-weight: bold; width: 200px; }\n" +
            ".image-container { margin: 10px 0; text-align: center; }\n" +
            ".report-image { max-width: 100%; border: 1px solid #ddd; }\n" +
            ".observations { padding: 15px; background: #f5f5f5; border-radius: 5px; min-height: 100px; }\n" +
            ".footer { margin-top: 30px; text-align: center; color: #666; font-size: 12px; }\n";
    }

    // ===================== SAVE REPORT =====================

    public static void saveReport(Context context, ReportData data, String outputPath) throws Exception {
        String html = generateHtmlReport(data);

        FileWriter writer = new FileWriter(outputPath);
        writer.write(html);
        writer.close();
    }
}
```

---

# 13. MODULO: SETTINGS

## 13.1 SettingsManager.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.content.*;
import android.content.SharedPreferences;

public class SettingsManager {

    private static final String PREFS_NAME = "bytec_us_pro_settings";
    private SharedPreferences prefs;
    private SharedPreferences.Editor editor;

    // Keys
    public static final String KEY_LANGUAGE = "language";
    public static final String KEY_THEME = "theme";
    public static final String KEY_AUTO_SAVE = "auto_save";
    public static final String KEY_SHOW_SCALE = "show_scale";
    public static final String KEY_SHOW_ANNOTATIONS = "show_annotations";
    public static final String KEY_CAPTURE_SOUND = "capture_sound";
    public static final String KEY_STORAGE_PATH = "storage_path";
    public static final String KEY_MAX_IMAGES = "max_images";

    // DICOM Keys
    public static final String KEY_DICOM_LOCAL_AE = "dicom_local_ae";
    public static final String KEY_DICOM_REMOTE_AE = "dicom_remote_ae";
    public static final String KEY_DICOM_HOST = "dicom_host";
    public static final String KEY_DICOM_PORT = "dicom_port";
    public static final String KEY_DICOM_MWL_AE = "dicom_mwl_ae";
    public static final String KEY_DICOM_MWL_HOST = "dicom_mwl_host";
    public static final String KEY_DICOM_MWL_PORT = "dicom_mwl_port";

    // Probe Keys
    public static final String KEY_PROBE_IP = "probe_ip";
    public static final String KEY_PROBE_TYPE = "probe_type";
    public static final String KEY_LAST_EXAM = "last_exam";

    // Singleton
    private static SettingsManager instance;

    public static synchronized SettingsManager getInstance(Context context) {
        if (instance == null) {
            instance = new SettingsManager(context);
        }
        return instance;
    }

    private SettingsManager(Context context) {
        prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        editor = prefs.edit();
    }

    // ===================== GENERAL SETTINGS =====================

    public String getLanguage() {
        return prefs.getString(KEY_LANGUAGE, "en");
    }

    public void setLanguage(String language) {
        editor.putString(KEY_LANGUAGE, language).apply();
    }

    public boolean isAutoSave() {
        return prefs.getBoolean(KEY_AUTO_SAVE, true);
    }

    public void setAutoSave(boolean autoSave) {
        editor.putBoolean(KEY_AUTO_SAVE, autoSave).apply();
    }

    public boolean isShowScale() {
        return prefs.getBoolean(KEY_SHOW_SCALE, true);
    }

    public void setShowScale(boolean show) {
        editor.putBoolean(KEY_SHOW_SCALE, show).apply();
    }

    public boolean isShowAnnotations() {
        return prefs.getBoolean(KEY_SHOW_ANNOTATIONS, true);
    }

    public void setShowAnnotations(boolean show) {
        editor.putBoolean(KEY_SHOW_ANNOTATIONS, show).apply();
    }

    public boolean isCaptureSound() {
        return prefs.getBoolean(KEY_CAPTURE_SOUND, true);
    }

    public void setCaptureSound(boolean sound) {
        editor.putBoolean(KEY_CAPTURE_SOUND, sound).apply();
    }

    public String getStoragePath() {
        return prefs.getString(KEY_STORAGE_PATH, "/sdcard/ByteCUSPro/");
    }

    public void setStoragePath(String path) {
        editor.putString(KEY_STORAGE_PATH, path).apply();
    }

    public int getMaxImages() {
        return prefs.getInt(KEY_MAX_IMAGES, 1000);
    }

    public void setMaxImages(int max) {
        editor.putInt(KEY_MAX_IMAGES, max).apply();
    }

    // ===================== DICOM SETTINGS =====================

    public String getDicomLocalAe() {
        return prefs.getString(KEY_DICOM_LOCAL_AE, "BYTEC_US");
    }

    public void setDicomLocalAe(String ae) {
        editor.putString(KEY_DICOM_LOCAL_AE, ae).apply();
    }

    public String getDicomRemoteAe() {
        return prefs.getString(KEY_DICOM_REMOTE_AE, "PACS");
    }

    public void setDicomRemoteAe(String ae) {
        editor.putString(KEY_DICOM_REMOTE_AE, ae).apply();
    }

    public String getDicomHost() {
        return prefs.getString(KEY_DICOM_HOST, "192.168.1.100");
    }

    public void setDicomHost(String host) {
        editor.putString(KEY_DICOM_HOST, host).apply();
    }

    public int getDicomPort() {
        return prefs.getInt(KEY_DICOM_PORT, 11112);
    }

    public void setDicomPort(int port) {
        editor.putInt(KEY_DICOM_PORT, port).apply();
    }

    public String getDicomMwlAe() {
        return prefs.getString(KEY_DICOM_MWL_AE, "MWL");
    }

    public void setDicomMwlAe(String ae) {
        editor.putString(KEY_DICOM_MWL_AE, ae).apply();
    }

    public String getDicomMwlHost() {
        return prefs.getString(KEY_DICOM_MWL_HOST, "192.168.1.101");
    }

    public void setDicomMwlHost(String host) {
        editor.putString(KEY_DICOM_MWL_HOST, host).apply();
    }

    public int getDicomMwlPort() {
        return prefs.getInt(KEY_DICOM_MWL_PORT, 11113);
    }

    public void setDicomMwlPort(int port) {
        editor.putInt(KEY_DICOM_MWL_PORT, port).apply();
    }

    // ===================== PROBE SETTINGS =====================

    public String getProbeIp() {
        return prefs.getString(KEY_PROBE_IP, "192.168.1.1");
    }

    public void setProbeIp(String ip) {
        editor.putString(KEY_PROBE_IP, ip).apply();
    }

    public String getProbeType() {
        return prefs.getString(KEY_PROBE_TYPE, "LINEAR");
    }

    public void setProbeType(String type) {
        editor.putString(KEY_PROBE_TYPE, type).apply();
    }

    public String getLastExam() {
        return prefs.getString(KEY_LAST_EXAM, "abdomen");
    }

    public void setLastExam(String exam) {
        editor.putString(KEY_LAST_EXAM, exam).apply();
    }

    // ===================== UTILS =====================

    public void clear() {
        editor.clear().apply();
    }

    public void resetToDefaults() {
        editor.clear();
        editor.apply();
    }
}
```

---

# 14. MODULO: SEGURIDAD

## 14.1 EncryptionUtil.smali (Referencia Java)

```java
package com.bytec.uspro;

import android.content.Context;
import android.security.keystore.KeyGenParameterSpec;
import android.security.keystore.KeyProperties;
import java.security.*;
import javax.crypto.*;
import javax.crypto.spec.*;

public class EncryptionUtil {

    private static final String ANDROID_KEYSTORE = "AndroidKeyStore";
    private static final String KEY_ALIAS = "ByteCUSProKey";
    private static final String TRANSFORMATION = "AES/GCM/NoPadding";

    // ===================== KEY GENERATION =====================

    public static SecretKey generateKey() throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEYSTORE);

        KeyGenParameterSpec spec = new KeyGenParameterSpec.Builder(
            KEY_ALIAS,
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .setKeySize(256)
            .build();

        keyGenerator.init(spec);
        return keyGenerator.generateKey();
    }

    // ===================== ENCRYPTION =====================

    public static byte[] encrypt(byte[] data) throws Exception {
        SecretKey key = getOrCreateKey();

        Cipher cipher = Cipher.getInstance(TRANSFORMATION);
        cipher.init(Cipher.ENCRYPT_MODE, key);

        byte[] iv = cipher.getIV();
        byte[] encrypted = cipher.doFinal(data);

        // Prepend IV to encrypted data
        byte[] result = new byte[iv.length + encrypted.length];
        System.arraycopy(iv, 0, result, 0, iv.length);
        System.arraycopy(encrypted, 0, result, iv.length, encrypted.length);

        return result;
    }

    // ===================== DECRYPTION =====================

    public static byte[] decrypt(byte[] encryptedData) throws Exception {
        SecretKey key = getOrCreateKey();

        // Extract IV
        byte[] iv = new byte[12]; // GCM IV length
        System.arraycopy(encryptedData, 0, iv, 0, iv.length);

        // Extract encrypted data
        byte[] data = new byte[encryptedData.length - iv.length];
        System.arraycopy(encryptedData, iv.length, data, 0, data.length);

        Cipher cipher = Cipher.getInstance(TRANSFORMATION);
        GCMParameterSpec spec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, key, spec);

        return cipher.doFinal(data);
    }

    // ===================== KEY MANAGEMENT =====================

    private static SecretKey getOrCreateKey() throws Exception {
        try {
            KeyStore keyStore = KeyStore.getInstance(ANDROID_KEYSTORE);
            keyStore.load(null);

            if (keyStore.containsAlias(KEY_ALIAS)) {
                return (SecretKey) keyStore.getKey(KEY_ALIAS, null);
            }
        } catch (Exception e) {
            // Fall through to generate new key
        }

        return generateKey();
    }

    // ===================== FILE ENCRYPTION =====================

    public static void encryptFile(String inputPath, String outputPath) throws Exception {
        java.io.FileInputStream fis = new java.io.FileInputStream(inputPath);
        java.io.FileOutputStream fos = new java.io.FileOutputStream(outputPath);

        byte[] data = new byte[(int) new java.io.File(inputPath).length()];
        fis.read(data);
        fis.close();

        byte[] encrypted = encrypt(data);
        fos.write(encrypted);
        fos.close();
    }

    public static void decryptFile(String inputPath, String outputPath) throws Exception {
        java.io.FileInputStream fis = new java.io.FileInputStream(inputPath);
        java.io.FileOutputStream fos = new java.io.FileOutputStream(outputPath);

        byte[] data = new byte[(int) new java.io.File(inputPath).length()];
        fis.read(data);
        fis.close();

        byte[] decrypted = decrypt(data);
        fos.write(decrypted);
        fos.close();
    }

    // ===================== HASHING =====================

    public static String hashPassword(String password) throws Exception {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(password.getBytes("UTF-8"));

        StringBuilder hexString = new StringBuilder();
        for (byte b : hash) {
            String hex = Integer.toHexString(0xff & b);
            if (hex.length() == 1) hexString.append('0');
            hexString.append(hex);
        }
        return hexString.toString();
    }
}
```

---

# 15. BUILD Y DEPLOY

## 15.1 build.sh (Script Completo)

```bash
#!/bin/bash
# ============================================================
# BYTEC US PRO - BUILD SCRIPT
# Version: 2.0.0
# ============================================================

set -e

# ===================== CONFIGURACION =====================
PROJECT_NAME="ByteCUSPro"
VERSION="2.0.0"
BUILD_TYPE="release"  # debug o release

# Paths
ORIGINAL_APK="original/dolphin/DolphinUSG.apk.1.1.1"
OUTPUT_DIR="output"
MODIFIED_DIR="modified"
BUILD_DIR="build"
KEYSTORE="signing/bytec.keystore"
ALIAS="bytec"
STOREPASS="ByteC2024!"
KEYPASS="ByteC2024!"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ===================== FUNCIONES =====================

print_banner() {
    echo -e "${BLUE}"
    echo "╔══════════════════════════════════════════════╗"
    echo "║       BYTEC US PRO - BUILD SYSTEM           ║"
    echo "║       Version: $VERSION                      ║"
    echo "╚══════════════════════════════════════════════╝"
    echo -e "${NC}"
}

print_step() {
    echo -e "\n${YELLOW}[$1/7] $2${NC}"
}

print_success() {
    echo -e "${GREEN}✓ $1${NC}"
}

print_error() {
    echo -e "${RED}✗ $1${NC}"
    exit 1
}

check_dependencies() {
    echo -e "\n${BLUE}Checking dependencies...${NC}"

    command -v java >/dev/null 2>&1 || print_error "Java not found"
    command -v apktool >/dev/null 2>&1 || print_error "apktool not found"
    command -v jarsigner >/dev/null 2>&1 || print_error "jarsigner not found"
    command -v zipalign >/dev/null 2>&1 || print_error "zipalign not found"
    command -v adb >/dev/null 2>&1 || print_error "adb not found"

    print_success "All dependencies found"
}

# ===================== FASES =====================

phase1_decompile() {
    print_step 1 "Decompiling original APK"

    if [ ! -d "$MODIFIED_DIR/smali" ]; then
        apktool d "$ORIGINAL_APK" -o "$MODIFIED_DIR" -f
        print_success "APK decompiled"
    else
        print_success "Already decompiled"
    fi
}

phase2_rebrand() {
    print_step 2 "Applying rebrand"

    # Update package name in smali
    echo "  Updating package references..."
    find "$MODIFIED_DIR/smali" -name "*.smali" -exec sed -i \
        's/com\/sonoptek\/wirelessusg3/com\/bytec\/uspro/g' {} + 2>/dev/null || true

    find "$MODIFIED_DIR/smali" -name "*.smali" -exec sed -i \
        's/com\/sonoptek/com\/bytec/g' {} + 2>/dev/null || true

    # Create new smali directory
    mkdir -p "$MODIFIED_DIR/smali/com/bytec/uspro"

    # Copy original smali
    if [ -d "$MODIFIED_DIR/smali/com/sonoptek/wirelessusg3" ]; then
        cp -r "$MODIFIED_DIR/smali/com/sonoptek/wirelessusg3/"* \
            "$MODIFIED_DIR/smali/com/bytec/uspro/" 2>/dev/null || true
    fi

    print_success "Rebrand applied"
}

phase3_resources() {
    print_step 3 "Updating resources"

    # Update strings
    if [ -f "$MODIFIED_DIR/res/values/strings.xml" ]; then
        sed -i 's/DolphinUSG/ByteC US Pro/g' "$MODIFIED_DIR/res/values/strings.xml"
        sed -i 's/Sonostar/ByteC/g' "$MODIFIED_DIR/res/values/strings.xml"
        sed -i 's/com\.sonoptek/com.bytec/g' "$MODIFIED_DIR/res/values/strings.xml"
    fi

    # Update colors
    if [ -f "$MODIFIED_DIR/res/values/colors.xml" ]; then
        # ByteC Blue: #1E88E5
        sed -i 's/#FF0000/#1E88E5/g' "$MODIFIED_DIR/res/values/colors.xml"
        sed -i 's/#00FF00/#00E676/g' "$MODIFIED_DIR/res/values/colors.xml"
    fi

    # Update manifest
    if [ -f "$MODIFIED_DIR/AndroidManifest.xml" ]; then
        sed -i 's/com\.sonoptek\.wirelessusg3/com.bytec.uspro/g' "$MODIFIED_DIR/AndroidManifest.xml"
    fi

    # Update icons
    if [ -f "assets/splash/bytec_logo.png" ]; then
        cp "assets/splash/bytec_logo.png" "$MODIFIED_DIR/res/mipmap-xxhdpi/ic_launcher.png"
        cp "assets/splash/bytec_logo.png" "$MODIFIED_DIR/res/mipmap-xxxhdpi/ic_launcher.png"
    fi

    print_success "Resources updated"
}

phase4_build() {
    print_step 4 "Building APK"

    mkdir -p "$OUTPUT_DIR"

    # Build with apktool
    apktool b "$MODIFIED_DIR" -o "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}-unsigned.apk"

    print_success "APK built"
}

phase5_sign() {
    print_step 5 "Signing APK"

    # Generate keystore if not exists
    if [ ! -f "$KEYSTORE" ]; then
        mkdir -p signing
        keytool -genkey -v \
            -keystore "$KEYSTORE" \
            -alias "$ALIAS" \
            -keyalg RSA \
            -keysize 2048 \
            -validity 10000 \
            -storepass "$STOREPASS" \
            -keypass "$KEYPASS" \
            -dname "CN=ByteC Medical, OU=Development, O=ByteC Medical Devices, L=Buenos Aires, ST=BA, AR=AR"
    fi

    # Sign
    jarsigner -verbose \
        -sigalg SHA1withRSA \
        -digestalg SHA1 \
        -keystore "$KEYSTORE" \
        -storepass "$STOREPASS" \
        -keypass "$KEYPASS" \
        "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}-unsigned.apk" \
        "$ALIAS"

    print_success "APK signed"
}

phase6_align() {
    print_step 6 "Aligning APK"

    zipalign -v 4 \
        "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}-unsigned.apk" \
        "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}.apk"

    print_success "APK aligned"
}

phase7_verify() {
    print_step 7 "Verifying APK"

    # Verify package name
    aapt dump badging "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}.apk" | grep "package:" | head -1

    # Check for errors
    aapt dump badging "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}.apk" > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        print_success "APK verification passed"
    else
        print_error "APK verification failed"
    fi

    # Clean up unsigned
    rm -f "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}-unsigned.apk"
    rm -f "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}-unsigned.apk.dsf"
    rm -f "$OUTPUT_DIR/${PROJECT_NAME}-${VERSION}.apk.dsf"
}

# ===================== MAIN =====================

main() {
    print_banner

    check_dependencies

    phase1_decompile
    phase2_rebrand
    phase3_resources
    phase4_build
    phase5_sign
    phase6_align
    phase7_verify

    echo -e "\n${GREEN}╔══════════════════════════════════════════════╗"
    echo -e "║       BUILD COMPLETED SUCCESSFULLY!         ║"
    echo -e "╚══════════════════════════════════════════════╝${NC}"
    echo -e "\nAPK: ${OUTPUT_DIR}/${PROJECT_NAME}-${VERSION}.apk"
    echo -e "\nTo install:"
    echo -e "  adb install ${OUTPUT_DIR}/${PROJECT_NAME}-${VERSION}.apk"
}

# Run
main "$@"
```

## 15.2 install.sh

```bash
#!/bin/bash
# BYTEC US PRO - INSTALL SCRIPT

APK="output/ByteCUSPro-2.0.0.apk"

echo "Uninstalling old version..."
adb uninstall com.sonoptek.wirelessusg3 2>/dev/null || true
adb uninstall com.bytec.uspro 2>/dev/null || true

echo "Installing ByteC US Pro..."
adb install -r "$APK"

echo "Verifying installation..."
adb shell pm list packages | grep bytec

echo "Done!"
```

## 15.3 clean.sh

```bash
#!/bin/bash
# BYTEC US PRO - CLEAN SCRIPT

echo "Cleaning build artifacts..."
rm -rf build/
rm -rf output/
rm -f modified/apktool.yml

echo "Clean complete!"
```

---

# 16. TESTING

## 16.1 test_dicom.sh

```bash
#!/bin/bash
# DICOM TEST SCRIPT

echo "=== DICOM Tests ==="

# Test 1: C-ECHO
echo "Test 1: C-ECHO"
adb shell am start -n com.bytec.uspro/.DicomActivity \
    --es action echo_test

sleep 5
adb shell cat /data/data/com.bytec.uspro/shared_prefs/*.xml | grep dicom_echo

# Test 2: C-STORE
echo "Test 2: C-STORE"
adb shell am start -n com.bytec.uspro/.DicomActivity \
    --es action store_test

sleep 10

# Test 3: MWL
echo "Test 3: MWL Query"
adb shell am start -n com.bytec.uspro/.WorklistActivity \
    --es action query_test

sleep 10

echo "=== DICOM Tests Complete ==="
```

## 16.2 test_connection.sh

```bash
#!/bin/bash
# CONNECTION TEST SCRIPT

echo "=== Connection Tests ==="

# Test 1: Probe Discovery
echo "Test 1: Probe Discovery"
adb shell am broadcast -a com.bytec.uspro.DISCOVER_PROBES

sleep 5

# Test 2: WiFi Connection
echo "Test 2: WiFi Connection"
adb shell am start -n com.bytec.uspro/.MainActivity \
    --es probe_ip 192.168.1.1

sleep 10

# Test 3: Frame Streaming
echo "Test 3: Frame Streaming"
adb shell am broadcast -a com.bytec.uspro.START_STREAM

sleep 10

# Test 4: Disconnect
echo "Test 4: Disconnect"
adb shell am broadcast -a com.bytec.uspro.DISCONNECT

echo "=== Connection Tests Complete ==="
```

---

# 17. REGULACIONES

## 17.1 IEC 62304 Compliance

| Requirement | Status | Notes |
|-------------|--------|-------|
| Software Safety Classification | Class B | Medical device software |
| Software Development Planning | Required | Document SDP |
| Software Requirements Analysis | Required | SRS document |
| Software Architectural Design | Required | SAD document |
| Software Detailed Design | Required | SDD document |
| Software Unit Implementation | In Progress | Code complete |
| Software Unit Verification | Required | Unit tests |
| Software Integration | Required | Integration tests |
| Software System Testing | Required | System tests |
| Software Release | Required | Release notes |
| Software Maintenance | Required | Maintenance plan |

## 17.2 FDA 21 CFR Part 820

| Requirement | Status |
|-------------|--------|
| Quality Management System | Required |
| Design Controls | Required |
| Document Controls | Required |
| Purchasing Controls | Required |
| Production and Process Controls | Required |
| Corrective and Preventive Actions | Required |
| Records Management | Required |

## 17.3 CE Marking (MDR)

| Requirement | Status |
|-------------|--------|
| Technical Documentation | Required |
| Conformity Assessment | Required |
| Clinical Evaluation | Required |
| Post-Market Surveillance | Required |
| UDI (Unique Device Identification) | Required |

---

# 18. APPENDICES

## A. DICOM Transfer Syntaxes Supported

| UID | Name | Status |
|-----|------|--------|
| 1.2.840.10008.1.2 | Implicit VR Little Endian | Supported |
| 1.2.840.10008.1.2.1 | Explicit VR Little Endian | Supported |
| 1.2.840.10008.1.2.4.50 | JPEG Baseline | Supported |
| 1.2.840.10008.1.2.4.51 | JPEG Extended | Supported |
| 1.2.840.10008.1.2.4.57 | JPEG Lossless | Supported |
| 1.2.840.10008.1.2.4.70 | JPEG Lossless First-Order | Supported |
| 1.2.840.10008.1.2.4.80 | JPEG-LS Lossless | Supported |
| 1.2.840.10008.1.2.4.90 | JPEG 2000 Lossless | Supported |
| 1.2.840.10008.1.2.4.91 | JPEG 2000 Lossy | Supported |

## B. Examination Presets

### Linear Probe
| Exam | Gain | DR | Enhance | Harmonic | Focus |
|------|------|-----|---------|----------|-------|
| Thyroid | 80 | 70 | 2 | ON | 2 |
| Small Parts | 80 | 70 | 2 | ON | 1 |
| Vascular | 80 | 70 | 2 | ON | 2 |
| Carotid | 80 | 70 | 2 | ON | 2 |
| MSK | 80 | 80 | 2 | OFF | 1 |

### Convex Probe
| Exam | Gain | DR | Enhance | Harmonic | Focus |
|------|------|-----|---------|----------|-------|
| Abdomen | 80 | 80 | 2 | OFF | 2 |
| Obstetric | 80 | 80 | 2 | ON | 1 |
| Cardiac | 80 | 80 | 2 | ON | 2 |
| Urology | 80 | 80 | 2 | ON | 1 |

## C. Error Codes

| Code | Description |
|------|-------------|
| E001 | Probe not found |
| E002 | Connection timeout |
| E003 | Invalid frame data |
| E004 | DICOM association failed |
| E005 | C-STORE failed |
| E006 | C-FIND failed |
| E007 | Patient not selected |
| E008 | No images to export |
| E009 | Storage permission denied |
| E010 | Encryption error |

## D. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-01-01 | Initial release |
| 1.1.0 | 2024-02-15 | Added Color Doppler |
| 1.2.0 | 2024-03-20 | Added DICOM MWL |
| 2.0.0 | 2024-07-05 | ByteC rebrand, splash, new features |

## E. Contacts

| Role | Email |
|------|-------|
| Technical Support | support@bytecmedical.com |
| Sales | sales@bytecmedical.com |
| Regulatory | regulatory@bytecmedical.com |

---

# FIN DEL DOCUMENTO

**ByteC US Pro v2.0.0 - Specificacion Tecnica Completa**
**© 2024 ByteC Medical Devices**
