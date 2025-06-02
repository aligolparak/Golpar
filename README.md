// اپلیکیشن ساده اندروید (Kotlin) برای تبدیل متن آهنگ به موزیک ویدیو // این نسخه فقط UI است و با سرور هوش مصنوعی فرضی در ارتباط خواهد بود

package com.example.lyricsvideo

import android.os.Bundle import android.widget.Button import android.widget.EditText import android.widget.Toast import android.widget.VideoView import androidx.appcompat.app.AppCompatActivity import kotlinx.coroutines.CoroutineScope import kotlinx.coroutines.Dispatchers import kotlinx.coroutines.launch import okhttp3.MediaType.Companion.toMediaTypeOrNull import okhttp3.OkHttpClient import okhttp3.Request import okhttp3.RequestBody.Companion.toRequestBody import org.json.JSONObject

class MainActivity : AppCompatActivity() {

private lateinit var lyricsInput: EditText
private lateinit var generateButton: Button
private lateinit var videoView: VideoView

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    lyricsInput = findViewById(R.id.lyricsInput)
    generateButton = findViewById(R.id.generateButton)
    videoView = findViewById(R.id.videoView)

    generateButton.setOnClickListener {
        val lyrics = lyricsInput.text.toString().trim()
        if (lyrics.isNotEmpty()) {
            generateMusicVideo(lyrics)
        } else {
            Toast.makeText(this, "لطفا متن آهنگ را وارد کنید.", Toast.LENGTH_SHORT).show()
        }
    }
}

private fun generateMusicVideo(lyrics: String) {
    CoroutineScope(Dispatchers.IO).launch {
        try {
            val json = JSONObject()
            json.put("lyrics", lyrics)

            val body = json.toString().toRequestBody("application/json".toMediaTypeOrNull())
            val client = OkHttpClient()

            val request = Request.Builder()
                .url("https://your-server.com/generate-video") // <-- آدرس سرور شما
                .post(body)
                .build()

            val response = client.newCall(request).execute()
            val responseBody = response.body?.string()

            // فرض: سرور یک لینک مستقیم به فایل ویدیو برمی‌گرداند
            if (response.isSuccessful && responseBody != null) {
                runOnUiThread {
                    videoView.setVideoPath(responseBody)
                    videoView.start()
                }
            } else {
                runOnUiThread {
                    Toast.makeText(this@MainActivity, "خطا در دریافت ویدیو.", Toast.LENGTH_SHORT).show()
                }
            }
        } catch (e: Exception) {
            runOnUiThread {
                Toast.makeText(this@MainActivity, "خطا: ${e.message}", Toast.LENGTH_LONG).show()
            }
        }
    }
}

}

