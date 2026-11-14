package com.vandijital.ekrantakip

import android.app.*
import android.content.Context
import android.content.Intent
import android.os.Build
import android.os.Bundle
import android.os.IBinder
import android.widget.*
import org.json.JSONObject
import java.io.BufferedReader
import java.io.InputStreamReader
import java.net.HttpURLConnection
import java.net.URL
import java.net.URLEncoder

class MainActivity : Activity() {
    override fun onCreate(s: Bundle?) {
        super.onCreate(s)
        val prefs = getSharedPreferences("cfg", Context.MODE_PRIVATE)
        val root = LinearLayout(this); root.orientation = LinearLayout.VERTICAL
        root.setPadding(48, 64, 48, 48)
        val t = TextView(this); t.text = "Ekran Takip"; t.textSize = 24f; root.addView(t)
        val info = TextView(this)
        info.text = "Token'i yapistir, Baslat'a bas. Sonra Telegram'da /ping yaz."
        info.setPadding(0, 24, 0, 24); root.addView(info)
        val token = EditText(this); token.hint = "Bot token"
        token.setText(prefs.getString("token", "")); root.addView(token)
        val start = Button(this); start.text = "Baslat"; root.addView(start)
        val stop = Button(this); stop.text = "Durdur"; root.addView(stop)
        val st = TextView(this); st.setPadding(0, 24, 0, 0); root.addView(st)
        setContentView(root)
        if (Build.VERSION.SDK_INT >= 33)
            requestPermissions(arrayOf(android.Manifest.permission.POST_NOTIFICATIONS), 1)
        start.setOnClickListener {
            val tk = token.text.toString().trim()
            if (tk.isEmpty()) { Toast.makeText(this, "Token bos", Toast.LENGTH_SHORT).show() }
            else {
                prefs.edit().putString("token", tk).apply()
                val i = Intent(this, BotService::class.java)
                if (Build.VERSION.SDK_INT >= 26) startForegroundService(i) else startService(i)
                st.text = "Servis basladi. Telegram'da /ping yaz."
            }
        }
        stop.setOnClickListener { stopService(Intent(this, BotService::class.java)); st.text = "Durduruldu." }
    }
}

class BotService : Service() {
    @Volatile private var running = false
    private var token = ""
    override fun onBind(i: Intent?): IBinder? = null
    override fun onCreate() { super.onCreate(); foreground() }
    override fun onStartCommand(i: Intent?, f: Int, id: Int): Int {
        token = getSharedPreferences("cfg", Context.MODE_PRIVATE).getString("token", "") ?: ""
        if (!running && token.isNotEmpty()) { running = true; Thread { loop() }.start() }
        return START_STICKY
    }
    private fun foreground() {
        val ch = "ekrantakip"
        if (Build.VERSION.SDK_INT >= 26)
            getSystemService(NotificationManager::class.java)
                .createNotificationChannel(NotificationChannel(ch, "Ekran Takip", NotificationManager.IMPORTANCE_LOW))
        val n = Notification.Builder(this, ch).setContentTitle("Ekran Takip calisiyor")
            .setContentText("Telegram botu aktif").setSmallIcon(android.R.drawable.ic_menu_view).build()
        startForeground(1, n)
    }
    private fun loop() {
        var offset = 0L
        while (running) {
            try {
                val txt = get("https://api.telegram.org/bot$token/getUpdates?timeout=50&offset=$offset") ?: continue
                val j = JSONObject(txt)
                if (!j.optBoolean("ok")) { Thread.sleep(2000); continue }
                val arr = j.getJSONArray("result")
                for (k in 0 until arr.length()) {
                    val u = arr.getJSONObject(k); offset = u.getLong("update_id") + 1
                    val m = u.optJSONObject("message") ?: continue
                    val chat = m.getJSONObject("chat").getLong("id")
                    val text = m.optString("text", "")
                    if (text.startsWith("/ping")) send(chat, "pong - calisiyorum")
                    else if (text.startsWith("/start")) send(chat, "Merhaba! Aktifim. /ping yaz.")
                }
            } catch (e: Exception) { try { Thread.sleep(3000) } catch (x: Exception) {} }
        }
    }
    private fun send(chat: Long, msg: String) =
        get("https://api.telegram.org/bot$token/sendMessage?chat_id=$chat&text=" + URLEncoder.encode(msg, "UTF-8"))
    private fun get(u: String): String? {
        var c: HttpURLConnection? = null
        return try {
            c = URL(u).openConnection() as HttpURLConnection
            c.connectTimeout = 15000; c.readTimeout = 60000
            val s = if (c.responseCode in 200..299) c.inputStream else c.errorStream
            BufferedReader(InputStreamReader(s)).use { it.readText() }
        } catch (e: Exception) { null } finally { c?.disconnect() }
    }
    override fun onDestroy() { running = false; super.onDestroy() }
}
