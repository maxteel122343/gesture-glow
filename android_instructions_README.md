# Guia de Integração Android (APK) - Modo Real

Para que o **AI Gesture Agent** funcione como uma sobreposição (overlay) no seu celular real, você precisará configurar um serviço nativo no Android Studio. O código web (React) será executado dentro desta sobreposição.

## 1. Permissões Necessárias (AndroidManifest.xml)

Adicione estas permissões no seu arquivo `app/src/main/AndroidManifest.xml`:

```xml
<!-- Permissão para desenhar sobre outros apps -->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<!-- Permissão para simular toques e ler a tela -->
<uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<application ...>
    <!-- Serviço de Sobreposição -->
    <service
        android:name=".OverlayService"
        android:enabled="true"
        android:exported="false" />

    <!-- Serviço de Acessibilidade (Opcional para Gestos Reais) -->
    <service
        android:name=".MyAccessibilityService"
        android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
        android:exported="true">
        <intent-filter>
            <action android:name="android.view.accessibility.AccessibilityService" />
        </intent-filter>
        <meta-data
            android:name="android.view.accessibility.accessibilityservice"
            android:resource="@xml/accessibility_service_config" />
    </service>
</application>
```

## 2. Lógica da Sidebar (OverlayService.kt)

Crie um arquivo `OverlayService.kt` para gerenciar a janela flutuante. Esta lógica garante que a sidebar fique sobreposta a outros aplicativos:

```kotlin
class OverlayService : Service() {
    private lateinit var windowManager: WindowManager
    private lateinit var sidebarView: View

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        showSidebar()
        return START_STICKY
    }

    private fun showSidebar() {
        windowManager = getSystemService(WINDOW_SERVICE) as WindowManager
        
        // Inflar a view da sidebar (seu layout XML com o WebView ou botões)
        sidebarView = LayoutInflater.from(this).inflate(R.layout.layout_sidebar, null)

        val params = WindowManager.LayoutParams(
            120, // Largura da sidebar (ajuste conforme necessário)
            WindowManager.LayoutParams.MATCH_PARENT,
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
                WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
            else
                WindowManager.LayoutParams.TYPE_PHONE,
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN,
            PixelFormat.TRANSLUCENT
        )

        params.gravity = Gravity.START or Gravity.TOP
        params.x = 0
        params.y = 0

        windowManager.addView(sidebarView, params)
    }

    override fun onDestroy() {
        super.onDestroy()
        if (::sidebarView.isInitialized) windowManager.removeView(sidebarView)
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

## 3. Como testar no Celular

1. **Build APK**: No Android Studio, vá em `Build > Build Bundle(s) / APK(s) > Build APK(s)`.
2. **Instalar**: Transfira o APK para o seu celular e instale.
3. **Permissões**: Ao abrir, o app deve solicitar "Aparecer sobre outros aplicativos". Você **deve** permitir manualmente nas configurações do Android.
4. **Iniciar**: Clique em "Vincular e Iniciar" no app para ativar o `OverlayService`.

O Sidebar aparecerá na lateral esquerda e permanecerá lá mesmo que você abra o Instagram, WhatsApp ou qualquer outro app.
