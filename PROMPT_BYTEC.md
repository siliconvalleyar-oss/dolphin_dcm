# ============================================================
# BYTEC ULTRASOUND - REBRAND & UPGRADE PROMPT
# ============================================================
# Rebrand de DolphinUSG → ByteC US Pro
# Empresa: ByteC Medical Devices
# Enfoque: Modificar APK existente + agregar features
# ============================================================

## CONTEXTO

Sos un senior Android developer especializado en medical devices.
Tenemos un APK decompileado de DolphinUSG (ultrasonido inalambrico).
Necesito rebrandearlo como "ByteC US Pro" y modernizarlo.

NO es una app nueva desde cero. Es modificar el APK existente,
reemplazar branding, agregar splash screen SVG, y mejorar features.

## OBJETIVOS

1. **Rebrand completo**: DolphinUSG → ByteC US Pro
2. **Splash screen moderno**: Animacion SVG al iniciar
3. **UI mejorada**: Colores, iconos, tipografia moderna
4. **Nuevas features**: Agregar funcionalidades escalables
5. **Mantener compatibilidad**: Que siga funcionando con el probe

## ESTRUCTURA DEL PROYECTO

```
bytec_us_pro/
├── original/                    # APK original decompileado
│   └── dolphin/                 # Directorio actual
├── modified/                    # APK modificado
│   ├── res/
│   │   ├── drawable/           # Nuevos iconos SVG/PNG
│   │   ├── values/
│   │   │   ├── strings.xml     # Textos en español/ingles
│   │   │   ├── colors.xml      # Colores ByteC
│   │   │   └── styles.xml      # Temas modernos
│   │   ├── layout/             # Layouts modificados
│   │   ├── anim/               # Animaciones
│   │   └── mipmap/             # Iconos launcher
│   ├── smali/                  # Codigo smali modificado
│   ├── assets/                 # Assets nuevos
│   │   ├── splash/
│   │   │   ├── logo.svg        # Logo ByteC
│   │   │   └── animation.json # Lottie animation
│   │   └── ...
│   └── AndroidManifest.xml     # Manifest modificado
├── tools/                      # Herramientas
│   ├── apktool.jar
│   ├── jadx/
│   └── signing/
├── build.sh                    # Script de build
└── PROMPT_BYTEC.md             # Este archivo
```

## PASO A PASO

### FASE 1: PREPARACION

#### 1.1 Decompilar el APK

```bash
# Decompilar con apktool
apktool d DolphinUSG.apk.1.1.1 -o modified -f

# Estructura resultante
modified/
├── AndroidManifest.xml
├── apktool.yml
├── lib/
├── original/
├── res/
└── smali/
```

#### 1.2 Analizar estructura

```bash
# Ver manifest
cat modified/AndroidManifest.xml

# Buscar clases principales
find modified/smali -name "*.smali" | head -20

# Buscar activities
grep -r "activity" modified/AndroidManifest.xml
```

### FASE 2: REBRAND

#### 2.1 Cambiar nombre de paquete

```xml
<!-- modified/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.bytec.uspro"  <!-- ANTES: com.sonoptek.wirelessusg3 -->
    android:versionCode="2"
    android:versionName="2.0.0">
```

#### 2.2 Renombrar directorios smali

```bash
# Mover smali de com/sonoptek a com/bytec
mkdir -p modified/smali/com/bytec/uspro
cp -r modified/smali/com/sonoptek/* modified/smali/com/bytec/uspro/

# Actualizar todas las referencias en smali
find modified/smali -name "*.smali" -exec sed -i \
  's/com\/sonoptek\/wirelessusg3/com\/bytec\/uspro/g' {} +

find modified/smali -name "*.smali" -exec sed -i \
  's/com\/sonoptek/com\/bytec/g' {} +
```

#### 2.3 Cambiar Application class

```bash
# En smali/com/bytec/uspro/MyApplication.smali
# Cambiar nombre de clase
sed -i 's/Lcom\/sonoptek\/wirelessusg3\/MyApplication;/Lcom\/bytec\/uspro\/MyApplication;/g' \
  modified/smali/com/bytec/uspro/MyApplication.smali
```

#### 2.4 Actualizar strings.xml

```xml
<!-- modified/res/values/strings.xml -->
<resources>
    <string name="app_name">ByteC US Pro</string>
    <string name="company_name">ByteC Medical Devices</string>
    <string name="copyright">© 2024 ByteC Medical. All rights reserved.</string>

    <!-- UI Strings -->
    <string name="mode_b">B-Mode</string>
    <string name="mode_m">M-Mode</string>
    <string name="mode_color">Color Doppler</string>
    <string name="mode_pw">PW Doppler</string>
    <string name="mode_biopsy">Biopsy Guide</string>

    <!-- Actions -->
    <string name="btn_freeze">Freeze</string>
    <string name="btn_capture">Capture</string>
    <string name="btn_measure">Measure</string>
    <string name="btn_export">Export DICOM</string>
    <string name="btn_report">Generate Report</string>

    <!-- Settings -->
    <string name="settings_title">Settings</string>
    <string name="settings_probe">Probe Connection</string>
    <string name="settings_dicom">DICOM Configuration</string>
    <string name="settings_patient">Patient Defaults</string>

    <!-- DICOM -->
    <string name="dicom_local_ae">Local AE Title</string>
    <string name="dicom_remote_ae">Remote AE Title</string>
    <string name="dicom_port">DICOM Port</string>
    <string name="dicom_mwl_ae">MWL AE Title</string>
    <string name="dicom_mwl_port">MWL Port</string>

    <!-- Messages -->
    <string name="connecting">Connecting to probe...</string>
    <string name="connected">Probe connected</string>
    <string name="disconnected">Probe disconnected</string>
    <string name="error_connection">Connection error</string>
    <string name="dicom_sending">Sending to PACS...</string>
    <string name="dicom_sent">Image sent successfully</string>
</resources>
```

#### 2.5 Cambiar colores corporativos

```xml
<!-- modified/res/values/colors.xml -->
<resources>
    <!-- ByteC Brand Colors -->
    <color name="bytec_primary">#1E88E5</color>        <!-- Azul ByteC -->
    <color name="bytec_primary_dark">#1565C0</color>
    <color name="bytec_accent">#00E676</color>          <!-- Verde accent -->
    <color name="bytec_background">#121212</color>      <!-- Dark mode -->
    <color name="bytec_surface">#1E1E1E</color>
    <color name="bytec_text_primary">#FFFFFF</color>
    <color name="bytec_text_secondary">#B0B0B0</color>

    <!-- Medical UI Colors -->
    <color name="ultrasound_background">#000000</color>
    <color name="ultrasound_grayscale">#808080</color>
    <color name="doppler_red">#FF1744</color>
    <color name="doppler_blue">#2979FF</color>
    <color name="pw_trace">#00E676</color>
    <color name="biopsy_guide">#FFEB3B</color>
    <color name="measurement_mark">#FF9100</color>

    <!-- Status Colors -->
    <color name="status_connected">#00E676</color>
    <color name="status_disconnected">#FF5252</color>
    <color name="status_frozen">#FFD740</color>
</resources>
```

### FASE 3: SPLASH SCREEN MODERNO

#### 3.1 Crear Splash Activity

```java
// modified/smali/com/bytec/uspro/SplashActivity.smali
// (o crear en Java y compilar)

public class SplashActivity extends AppCompatActivity {
    private ImageView logoView;
    private TextView companyText;
    private Animation fadeIn;
    private Animation slideUp;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Full screen
        getWindow().setFlags(
            WindowManager.LayoutParams.FLAG_FULLSCREEN,
            WindowManager.LayoutParams.FLAG_FULLSCREEN
        );

        setContentView(R.layout.activity_splash);

        // Animaciones
        fadeIn = AnimationUtils.loadAnimation(this, R.anim.fade_in);
        slideUp = AnimationUtils.loadAnimation(this, R.anim.slide_up);

        logoView = findViewById(R.id.splash_logo);
        companyText = findViewById(R.id.splash_company);

        // Animar logo
        logoView.startAnimation(fadeIn);

        // Animar texto despues
        new Handler().postDelayed(() -> {
            companyText.setVisibility(View.VISIBLE);
            companyText.startAnimation(slideUp);
        }, 500);

        // Ir a MainActivity despues de 3 segundos
        new Handler().postDelayed(() -> {
            Intent intent = new Intent(this, MainActivity.class);
            startActivity(intent);
            overridePendingTransition(R.anim.fade_in, R.anim.fade_out);
            finish();
        }, 3000);
    }
}
```

#### 3.2 Layout del Splash

```xml
<!-- modified/res/layout/activity_splash.xml -->
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/bytec_background"
    android:gravity="center">

    <!-- Logo ByteC -->
    <ImageView
        android:id="@+id/splash_logo"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true"
        android:src="@drawable/bytec_logo"
        android:alpha="0"/>

    <!-- Company Name -->
    <TextView
        android:id="@+id/splash_company"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/splash_logo"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="20dp"
        android:text="ByteC Medical"
        android:textColor="@color/bytec_text_primary"
        android:textSize="28sp"
        android:textStyle="bold"
        android:visibility="invisible"/>

    <!-- Version -->
    <TextView
        android:id="@+id/splash_version"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/splash_company"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="10dp"
        android:text="US Pro v2.0"
        android:textColor="@color/bytec_text_secondary"
        android:textSize="14sp"
        android:visibility="invisible"/>

    <!-- Loading indicator -->
    <ProgressBar
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="50dp"
        android:indeterminateTint="@color/bytec_accent"/>
</RelativeLayout>
```

#### 3.3 Animaciones

```xml
<!-- modified/res/anim/fade_in.xml -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator">
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="1000"/>
</set>

<!-- modified/res/anim/slide_up.xml -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator">
    <translate
        android:fromYDelta="30%"
        android:toYDelta="0%"
        android:duration="800"/>
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="800"/>
</set>

<!-- modified/res/anim/fade_out.xml -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha
        android:fromAlpha="1.0"
        android:toAlpha="0.0"
        android:duration="500"/>
</set>
```

#### 3.4 Logo ByteC SVG → PNG

```bash
# Convertir SVG a PNG para Android
# Usar ImageMagick o Inkscape

# Logo principal
inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/drawable/bytec_logo.png \
  -w 400 -h 400

# Iconos por densidad
inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/mipmap-mdpi/ic_launcher.png \
  -w 48 -h 48

inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/mipmap-hdpi/ic_launcher.png \
  -w 72 -h 72

inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/mipmap-xhdpi/ic_launcher.png \
  -w 96 -h 96

inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/mipmap-xxhdpi/ic_launcher.png \
  -w 144 -h 144

inkscape bytec_logo.svg --export-type=png \
  --export-filename=modified/res/mipmap-xxxhdpi/ic_launcher.png \
  -w 192 -h 192
```

#### 3.5 Actualizar Manifest para Splash

```xml
<!-- modified/AndroidManifest.xml -->
<activity
    android:name="com.bytec.uspro.SplashActivity"
    android:theme="@style/SplashTheme"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>

<activity
    android:name="com.bytec.uspro.MainActivity"
    android:theme="@style/AppTheme"
    android:screenOrientation="landscape"/>

<!-- Tema Splash -->
<style name="SplashTheme" parent="Theme.AppCompat.NoActionBar">
    <item name="android:windowBackground">@color/bytec_background</item>
    <item name="android:windowFullscreen">true</item>
</style>
```

### FASE 4: UI MODERNA

#### 4.1 Tema principal

```xml
<!-- modified/res/values/styles.xml -->
<resources>
    <!-- Base application theme -->
    <style name="AppTheme" parent="Theme.AppCompat.DayNight.NoActionBar">
        <item name="colorPrimary">@color/bytec_primary</item>
        <item name="colorPrimaryDark">@color/bytec_primary_dark</item>
        <item name="colorAccent">@color/bytec_accent</item>
        <item name="android:windowBackground">@color/bytec_background</item>
        <item name="android:textColorPrimary">@color/bytec_text_primary</item>
        <item name="android:textColorSecondary">@color/bytec_text_secondary</item>
    </style>

    <!-- Button Style -->
    <style name="ByteCButton" parent="Widget.AppCompat.Button.Colored">
        <item name="backgroundTint">@color/bytec_primary</item>
        <item name="android:textColor">@color/bytec_text_primary</item>
        <item name="cornerRadius">8dp</item>
    </style>

    <!-- Card Style -->
    <style name="ByteCCard" parent="Widget.AppCompat.CardView">
        <item name="cardBackgroundColor">@color/bytec_surface</item>
        <item name="cardCornerRadius">12dp</item>
        <item name="cardElevation">4dp</item>
    </style>
</resources>
```

#### 4.2 Iconos modernos

```xml
<!-- modified/res/drawable/ic_mode_b.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M12,2C6.48,2 2,6.48 2,12s4.48,10 10,10
        10,-4.48 10,-10S17.52,2 12,2zM12,20c-4.41,
        0 -8,-3.59 -8,-8s3.59,-8 8,-8 8,3.59 8,8
        -3.59,8 -8,8z"/>
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M12,6c-3.31,0 -6,2.69 -6,6s2.69,6
        6,6 6,-2.69 6,-6 -2.69,-6 -6,-6z"/>
</vector>

<!-- modified/res/drawable/ic_freeze.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M6,4h2v16H6zM16,4h2v16h-2zM9,10h6v4H9z"/>
</vector>

<!-- modified/res/drawable/ic_capture.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M12,12m-3.2,0a3.2,3.2 0,1 1,6.4
        0a3.2,3.2 0,1 1,-6.4 0"/>
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M9,2L7.17,4H4c-1.1,0 -2,0.9
        -2,2v12c0,1.1 0.9,2 2,2h16c1.1,0 2,-0.9
        2,-2V6c0,-1.1 -0.9,-2 -2,-2h-3.17L15,2H9z"/>
</vector>
```

### FASE 5: NUEVAS FEATURES

#### 5.1 Feature: Multi-Probe Support

```java
// modified/smali/com/bytec/uspro/ProbeManager.smali
// (concepto)

public class ProbeManager {
    private Map<String, ProbeConfig> probeConfigs;
    private ProbeConfig currentProbe;

    public ProbeManager() {
        probeConfigs = new HashMap<>();
        // Cargar configs desde assets
        loadProbeConfigs();
    }

    public void selectProbe(String probeId) {
        currentProbe = probeConfigs.get(probeId);
        applyProbeConfig();
    }

    public List<String> getAvailableProbes() {
        return new ArrayList<>(probeConfigs.keySet());
    }

    private void applyProbeConfig() {
        // Aplicar configuracion del probe seleccionado
        // Gain, DR, Focus, etc.
    }
}
```

#### 5.2 Feature: Patient Database

```java
// modified/smali/com/bytec/uspro/PatientDatabase.smali
// (concepto)

public class PatientDatabase {
    private static final String DB_NAME = "bytec_patients.db";
    private SQLiteDatabase db;

    public void init(Context context) {
        db = context.openOrCreateDatabase(DB_NAME,
            Context.MODE_PRIVATE, null);
        createTables();
    }

    private void createTables() {
        db.execSQL("CREATE TABLE IF NOT EXISTS patients (" +
            "id INTEGER PRIMARY KEY AUTOINCREMENT," +
            "name TEXT," +
            "birth_date TEXT," +
            "sex TEXT," +
            "phone TEXT," +
            "notes TEXT," +
            "created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP)");

        db.execSQL("CREATE TABLE IF NOT EXISTS studies (" +
            "id INTEGER PRIMARY KEY AUTOINCREMENT," +
            "patient_id INTEGER," +
            "study_date TEXT," +
            "description TEXT," +
            "images_count INTEGER," +
            "dicom_sent BOOLEAN DEFAULT 0," +
            "FOREIGN KEY(patient_id) REFERENCES patients(id))");
    }

    public long savePatient(Patient patient) {
        ContentValues values = new ContentValues();
        values.put("name", patient.name);
        values.put("birth_date", patient.birthDate);
        values.put("sex", patient.sex);
        return db.insert("patients", null, values);
    }

    public List<Patient> searchPatients(String query) {
        List<Patient> patients = new ArrayList<>();
        Cursor cursor = db.rawQuery(
            "SELECT * FROM patients WHERE name LIKE ?",
            new String[]{"%" + query + "%"});

        while (cursor.moveToNext()) {
            Patient p = new Patient();
            p.id = cursor.getLong(0);
            p.name = cursor.getString(1);
            patients.add(p);
        }
        cursor.close();
        return patients;
    }
}
```

#### 5.3 Feature: DICOM Worklist Integration

```java
// modified/smali/com/bytec/uspro/WorklistService.smali
// (concepto)

public class WorklistService {
    private DicomConnection connection;

    public WorklistService(String host, int port, String aeTitle) {
        connection = new DicomConnection(host, port, aeTitle);
    }

    public List<WorklistItem> queryWorklist(String patientName) {
        // C-FIND para Modality Worklist
        DicomDataset query = new DicomDataset();
        query.setString(Tag.PatientName, patientName);
        query.setString(Tag.Modality, "US");
        query.setString(Tag.ScheduledStationAET, "BYTEC_US");

        return connection.cfind(
            UID.ModalityWorklistInformationModelFIND,
            query
        );
    }

    public void sendMpps(WorklistItem item) {
        // MPPS (Modality Performed Procedure Step)
        DicomDataset dataset = new DicomDataset();
        dataset.setString(Tag.SOPClassUID,
            UID.ModalityPerformedProcedureStepSOPClass);
        dataset.setString(Tag.PerformedProcedureStepStatus,
            "COMPLETED");

        connection.nEventReport(dataset);
    }
}
```

#### 5.4 Feature: Image Annotations

```java
// modified/smali/com/bytec/uspro/AnnotationManager.smali
// (concepto)

public class AnnotationManager {
    private List<Annotation> annotations;

    public void addAnnotation(AnnotationType type, String text) {
        Annotation a = new Annotation();
        a.type = type;
        a.text = text;
        a.timestamp = System.currentTimeMillis();
        annotations.add(a);
    }

    public void addMeasurement(MeasurementType type,
                               Point p1, Point p2) {
        Measurement m = new Measurement();
        m.type = type;
        m.point1 = p1;
        m.point2 = p2;
        m.value = calculateValue(type, p1, p2);
        annotations.add(m);
    }

    public Bitmap exportWithAnnotations(Bitmap original) {
        Bitmap result = original.copy(Bitmap.Config.ARGB_8888, true);
        Canvas canvas = new Canvas(result);

        for (Annotation a : annotations) {
            drawAnnotation(canvas, a);
        }

        return result;
    }

    private double calculateValue(MeasurementType type,
                                  Point p1, Point p2) {
        switch (type) {
            case DISTANCE:
                return Point.distance(p1, p2);
            case AREA:
                return calculateArea(p1, p2);
            case ANGLE:
                return calculateAngle(p1, p2);
            default:
                return 0;
        }
    }
}
```

#### 5.5 Feature: Cloud Backup (Opcional)

```java
// modified/smali/com/bytec/uspro/CloudService.smali
// (concepto)

public class CloudService {
    private String apiKey;
    private String serverUrl;

    public void uploadStudy(Study study, Callback callback) {
        // Subir estudio a la nube ByteC
        new Thread(() -> {
            try {
                // Comprimir imagenes
                byte[] data = compressStudy(study);

                // Subir via HTTP POST
                HttpURLConnection conn = (HttpURLConnection)
                    new URL(serverUrl + "/api/studies").openConnection();
                conn.setRequestMethod("POST");
                conn.setRequestProperty("Authorization",
                    "Bearer " + apiKey);
                conn.setDoOutput(true);

                OutputStream os = conn.getOutputStream();
                os.write(data);
                os.close();

                int response = conn.getResponseCode();
                callback.onResult(response == 200);
            } catch (Exception e) {
                callback.onError(e.getMessage());
            }
        }).start();
    }
}
```

### FASE 6: RECOMPILAR

#### 6.1 Recompilar con apktool

```bash
# Recompilar
apktool b modified -o bytec_us_pro.apk

# Verificar
aapt dump badging bytec_us_pro.apk | grep package
```

#### 6.2 Generar keystore

```bash
# Generar keystore de firma
keytool -genkey -v \
  -keystore bytec.keystore \
  -alias bytec \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -storepass bytec123 \
  -keypass bytec123 \
  -dname "CN=ByteC Medical, OU=Development, O=ByteC, L=Buenos Aires, ST=BA, AR=AR"
```

#### 6.3 Firmar APK

```bash
# Firmar con jarsigner
jarsigner -verbose \
  -sigalg SHA1withRSA \
  -digestalg SHA1 \
  -keystore bytec.keystore \
  -storepass bytec123 \
  bytec_us_pro.apk \
  bytec

# Align con zipalign
zipalign -v 4 bytec_us_pro.apk bytec_us_pro_aligned.apk
```

#### 6.4 Instalar y probar

```bash
# Desinstalar version anterior
adb uninstall com.sonoptek.wirelessusg3

# Instalar nueva version
adb install bytec_us_pro_aligned.apk

# Verificar
adb shell pm list packages | grep bytec
```

## ARCHIVOS NECESARIOS

### Logo ByteC (SVG)

```svg
<!-- assets/splash/bytec_logo.svg -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#1E88E5;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#00E676;stop-opacity:1" />
    </linearGradient>
  </defs>

  <!-- Background circle -->
  <circle cx="100" cy="100" r="90" fill="url(#grad1)"/>

  <!-- B letter -->
  <text x="50%" y="55%" text-anchor="middle"
        font-family="Arial, sans-serif"
        font-size="80" font-weight="bold"
        fill="white">B</text>

  <!-- ByteC text -->
  <text x="50%" y="85%" text-anchor="middle"
        font-family="Arial, sans-serif"
        font-size="24" font-weight="bold"
        fill="white">ByteC</text>
</svg>
```

### Splash Animation (Lottie JSON)

```json
{
  "v": "5.7.4",
  "fr": 30,
  "ip": 0,
  "op": 60,
  "w": 200,
  "h": 200,
  "nm": "ByteC Logo",
  "ddd": 0,
  "assets": [],
  "layers": [
    {
      "ddd": 0,
      "ind": 1,
      "ty": 4,
      "nm": "Circle",
      "sr": 1,
      "ks": {
        "o": {"a": 0, "k": 100},
        "r": {"a": 0, "k": 0},
        "p": {"a": 0, "k": [100, 100, 0]},
        "a": {"a": 0, "k": [0, 0, 0]},
        "s": {
          "a": 1,
          "k": [
            {"t": 0, "s": [0, 0, 100], "e": [100, 100, 100]},
            {"t": 30, "s": [100, 100, 100]}
          ]
        }
      },
      "shapes": [
        {
          "ty": "el",
          "p": {"a": 0, "k": [0, 0]},
          "s": {"a": 0, "k": [180, 180]},
          "nm": "Ellipse"
        },
        {
          "ty": "fl",
          "c": {"a": 0, "k": [0.12, 0.53, 0.9, 1]},
          "o": {"a": 0, "k": 100},
          "nm": "Fill"
        }
      ]
    }
  ]
}
```

## SCRIPT DE BUILD

```bash
#!/bin/bash
# build.sh - Script de build para ByteC US Pro

set -e

echo "================================"
echo "  ByteC US Pro - Build Script"
echo "================================"

# Configuracion
APK_NAME="ByteCUSPro"
VERSION="2.0.0"
KEYSTORE="bytec.keystore"
ALIAS="bytec"
STOREPASS="bytec123"

# 1. Decompilar si es necesario
if [ ! -d "modified/smali" ]; then
    echo "[1/6] Decompilando APK original..."
    apktool d original/DolphinUSG.apk.1.1.1 -o modified -f
fi

# 2. Aplicar cambios
echo "[2/6] Aplicando modificaciones..."
# (Los cambios ya estan en modified/)

# 3. Recompilar
echo "[3/6] Recompilando APK..."
apktool b modified -o ${APK_NAME}.apk

# 4. Firmar
echo "[4/6] Firmando APK..."
jarsigner -verbose \
  -sigalg SHA1withRSA \
  -digestalg SHA1 \
  -keystore ${KEYSTORE} \
  -storepass ${STOREPASS} \
  ${APK_NAME}.apk \
  ${ALIAS}

# 5. Align
echo "[5/6] Aligning APK..."
zipalign -v 4 ${APK_NAME}.apk ${APK_NAME}_aligned.apk

# 6. Verificar
echo "[6/6] Verificando APK..."
aapt dump badging ${APK_NAME}_aligned.apk | grep package

echo ""
echo "================================"
echo "  Build completado!"
echo "  APK: ${APK_NAME}_aligned.apk"
echo "================================"
echo ""
echo "Para instalar:"
echo "  adb install ${APK_NAME}_aligned.apk"
```

## TESTING

### Pruebas de QA

```bash
# 1. Instalar
adb install bytec_us_pro_aligned.apk

# 2. Verificar nombre
adb shell pm list packages | grep bytec
# Output: package:com.bytec.uspro

# 3. Verificar splash
adb shell am start -n com.bytec.uspro/.SplashActivity

# 4. Verificar conexion probe
# (Probar con probe real)

# 5. Verificar DICOM
# (Probar con PACS server)

# 6. Verificar captura
# (Capturar imagen y verificar)

# 7. Verificar reporte
# (Generar reporte PDF)
```

## RESULTADO ESPERADO

| Aspecto | Original | Modificado |
|---------|----------|------------|
| Nombre | DolphinUSG | ByteC US Pro |
| Package | com.sonoptek.wirelessusg3 | com.bytec.uspro |
| Splash | Ninguno | Animacion SVG moderna |
| Colores | Generic | ByteC Brand (Azul/Verde) |
| Icono | Sonostar | ByteC |
| Features | Basic | Multi-probe, DB, Cloud |

## TIMELINE

| Fase | Duracion | Entregable |
|------|----------|------------|
| Fase 1: Setup | 1 dia | APK decompileado |
| Fase 2: Rebrand | 2 dias | Nombre, paquete, strings |
| Fase 3: Splash | 1 dia | Splash screen animado |
| Fase 4: UI | 2 dias | Colores, iconos, temas |
| Fase 5: Features | 5 dias | DB, Worklist, Cloud |
| Fase 6: Build | 1 dia | APK firmado |
| **Total** | **12 dias** | APK listo para distribuir |

## CONSIDERACIONES LEGALES

1. **Licencia**: Verificar que la app original permita rebranding
2. **DICOM**: Mantener compatibilidad con estandar
3. **Medical Device**: Cumplir regulaciones si es para uso clinico
4. **Datos Paciente**: Encriptar datos sensibles
5. **Update OTA**: Considerar sistema de actualizaciones
