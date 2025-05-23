# zazaza

package com.example.timerallinone

import android.annotation.SuppressLint
import android.app.TimePickerDialog
import android.media.MediaPlayer
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.selection.toggleable
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import kotlinx.coroutines.delay
import java.util.*

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AllInOneTimerApp()
        }
    }
}

@SuppressLint("DefaultLocale")
@Composable
fun AllInOneTimerApp() {
    val context = LocalContext.current
    val mediaPlayer = remember { MediaPlayer.create(context, R.raw.alarm_sound) }

    var stopwatchTime by remember { mutableIntStateOf(0) }
    var stopwatchRunning by remember { mutableStateOf(false) }

    var timerTime by remember { mutableIntStateOf(0) }
    var timerRunning by remember { mutableStateOf(false) }

    var alarmTime by remember { mutableStateOf("") }
    var alarmHour by remember { mutableIntStateOf(-1) }
    var alarmMinute by remember { mutableIntStateOf(-1) }

    val selectedDays = remember { mutableStateListOf<String>() }
    val daysOfWeek = listOf("Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс")

    val calendar = Calendar.getInstance()

    // Секундомер
    LaunchedEffect(stopwatchRunning) {
        while (stopwatchRunning) {
            delay(1000)
            stopwatchTime += 1
        }
    }

    // Таймер
    LaunchedEffect(timerRunning) {
        while (timerRunning && timerTime > 0) {
            delay(1000)
            timerTime -= 1
        }
        if (timerRunning && timerTime == 0) {
            timerRunning = false
            mediaPlayer.start()
        }
    }

    // Проверка срабатывания будильника
    LaunchedEffect(alarmHour, alarmMinute, selectedDays) {
        while (true) {
            delay(1000)
            val now = Calendar.getInstance()
            val currentHour = now.get(Calendar.HOUR_OF_DAY)
            val currentMinute = now.get(Calendar.MINUTE)
            val currentDay = now.get(Calendar.DAY_OF_WEEK) // Sunday = 1 ... Saturday = 7

            val dayMap = mapOf(
                Calendar.MONDAY to "Пн",
                Calendar.TUESDAY to "Вт",
                Calendar.WEDNESDAY to "Ср",
                Calendar.THURSDAY to "Чт",
                Calendar.FRIDAY to "Пт",
                Calendar.SATURDAY to "Сб",
                Calendar.SUNDAY to "Вс"
            )

            if (alarmHour == currentHour && alarmMinute == currentMinute) {
                val today = dayMap[currentDay]
                if (today in selectedDays) {
                    mediaPlayer.start()
                    delay(60000) // чтобы звук не проигрывался каждую секунду в течение минуты
                }
            }
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(24.dp)
    ) {
        // Секундомер
        Text("Секундомер: ${stopwatchTime} сек", style = MaterialTheme.typography.headlineSmall)
        Row(horizontalArrangement = Arrangement.spacedBy(16.dp)) {
            Button(onClick = { stopwatchRunning = true }) { Text("Старт") }
            Button(onClick = { stopwatchRunning = false }) { Text("Стоп") }
            Button(onClick = {
                stopwatchRunning = false
                stopwatchTime = 0
            }) { Text("Сброс") }
        }

        HorizontalDivider()

        // Таймер
        Text("Таймер: ${timerTime} сек", style = MaterialTheme.typography.headlineSmall)
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { timerTime += 10 }) { Text("+10 сек") }
            Button(onClick = { timerTime = 0 }) { Text("Сброс") }
        }
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { timerRunning = true }) { Text("Старт") }
            Button(onClick = { timerRunning = false }) { Text("Стоп") }
        }

        HorizontalDivider()

        // Будильник
        Text("Будильник: $alarmTime", style = MaterialTheme.typography.headlineSmall)

        Button(onClick = {
            val now = Calendar.getInstance()
            TimePickerDialog(
                context,
                { _, hour, minute ->
                    alarmHour = hour
                    alarmMinute = minute
                    alarmTime = String.format("%02d:%02d", hour, minute)
                },
                now.get(Calendar.HOUR_OF_DAY),
                now.get(Calendar.MINUTE),
                true
            ).show()
        }) {
            Text("Поставить будильник")
        }

        Text("Выберите дни недели:")

        val rows = daysOfWeek.chunked(4) // По 4 дня в строке
        rows.forEach { rowDays ->
            Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                rowDays.forEach { day ->
                    Row(
                        verticalAlignment = androidx.compose.ui.Alignment.CenterVertically,
                        modifier = Modifier
                            .padding(4.dp)
                            .weight(1f)
                            .toggleable(
                                value = selectedDays.contains(day),
                                onValueChange = {
                                    if (it) selectedDays.add(day) else selectedDays.remove(day)
                                }
                            )
                    ) {
                        Checkbox(
                            checked = selectedDays.contains(day),
                            onCheckedChange = null
                        )
                        Text(day)
                    }
                }
            }
        }
    }
}
