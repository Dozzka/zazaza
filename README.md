# zazaza

package com.example.timerallinone

import android.annotation.SuppressLint
import android.app.TimePickerDialog
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
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

    var stopwatchTime by remember { mutableIntStateOf(0) }
    var stopwatchRunning by remember { mutableStateOf(false) }

    var timerTime by remember { mutableIntStateOf(0) }
    var timerRunning by remember { mutableStateOf(false) }

    var alarmTime by remember { mutableStateOf("") }


    LaunchedEffect(stopwatchRunning) {
        while (stopwatchRunning) {
            delay(1000)
            stopwatchTime += 1
        }
    }


    LaunchedEffect(timerRunning) {
        while (timerRunning && timerTime > 0) {
            delay(1000)
            timerTime -= 1
        }
        if (timerRunning && timerTime == 0) {
            timerRunning = false
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp), verticalArrangement = Arrangement.spacedBy(24.dp)
    ) {
        Text("Секундомер: ${stopwatchTime}сек", style = MaterialTheme.typography.headlineSmall)
        Row(horizontalArrangement = Arrangement.spacedBy(16.dp)) {
            Button(onClick = { stopwatchRunning = true }) { Text("Старт") }
            Button(onClick = { stopwatchRunning = false }) { Text("Стоп") }
            Button(onClick = {
                stopwatchRunning = false
                stopwatchTime = 0
            }) { Text("Сброс") }
        }

        HorizontalDivider()

        Text("Таймер: ${timerTime}сек", style = MaterialTheme.typography.headlineSmall)
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { timerTime += 10 }) { Text("+10сек") }
            Button(onClick = { timerTime = 0 }) { Text("Сброс") }
        }
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { timerRunning = true }) { Text("Старт") }
            Button(onClick = { timerRunning = false }) { Text("Стоп") }
        }

        HorizontalDivider()

        Text("Будильник: $alarmTime", style = MaterialTheme.typography.headlineSmall)
        Button(onClick = {
            val calendar = Calendar.getInstance()
            TimePickerDialog(
                context,
                { _, hour, minute ->
                    alarmTime = String.format("%02d:%02d", hour, minute)
                },
                calendar.get(Calendar.HOUR_OF_DAY),
                calendar.get(Calendar.MINUTE),
                true
            ).show()
        }) {
            Text("Поставить будильник")
        }
    }
