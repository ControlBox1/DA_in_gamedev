# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #1 выполнил(а):
- Борнуков Егор Евгеньевич
- РИ230912
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * |    |
| Задание 2 | * |    |
| Задание 3 | * |    |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Ознакомиться с основными операторами зыка Python на примере реализации линейной регрессии.

## Задание 1
### Выберите одну из игровых переменных в игре СПАСТИ РТФ: Выживание (HP, SP, игровая валюта, здоровье и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.
Ход работы:

Для рассмотрения я выбрал деньги. По сути они отображают прогресс игрока. Они нужны для:
- Получения снаряжения
- Получения расходников
  
![изображение](https://github.com/user-attachments/assets/8a8c4db4-129d-4856-8ba0-9f3e8adcc720)


## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в игре “СПАСТИ РТФ:Выживание”. Средствами google-sheets визуализируйте данные в google-таблице (постройте график / диаграмму и пр.) для наглядного представления выбранной игровой величины. Опишите характер изменения этой величины, опишите недостатки в реализации этой величины (например, в игре может произойти условие наступления эксплойта) и предложите до 3-х вариантов модификации условий работы с переменной, чтобы сделать игровой опыт лучше.

Предположим игровую ситуацию - игрок проходит волну, зарабатывая 50 монет, из которых 1 тратит на улучшение, после чего покупает АК за 45 монет и умирает на следующей волне, зарабботав ещё 15 монет
Для генерации данного сценария используем скрипт:

```py
mport gspread
import numpy as np

values = []

def WriteData():
    global index
    sh.sheet1.update('A1', [[i] for i in range(len(values))])
    sh.sheet1.update('B1', [[i] for i in values])
    index += 1

gc = gspread.service_account(filename='unitydatascience-454407-c0f6a104c299.json')
sh = gc.open("MLTable")
i = 0
currency = 0
while i <= 51:
    i += 1
    # Покупка улучшения
    if i != 20:
        currency += 1
    else:
        currency -= 1
        
    values.append(currency)

# Покупка автомата в конце волны
currency -= 45
values.append(i)

while i <= 15:
    i += 1
    currency += 1
    values.append(currency)
    # Смерть
WriteData()
```

После выполнения получаем следующий результат:

![изображение](https://github.com/user-attachments/assets/e2d6a004-53d4-4c4d-99f7-e8d973a98b82)

Данные параметры можно улучшить, если:
- Давать  больше денег на более высоких волнах. Улучшения стоят дорого, но заработок не увеличивается
- Переработать цену улучшений. После первой волны я могу позолить себе половину улучшений, но вторую половину позволить себе я могу через нескеолько десятков волн
- Переработка цен на улучшения в середине игры. Вампиризм стоит 1 монету, хотя является субъективно лучшим улучшением. Это отбивает желание пробовать остальные, ведь оно позволит сохранить свои дорогие улучшеения на больший период времени

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

- Копируем код из методички, изменяя его так, чтобы он воспроизводил "Good" когда у нас растёт казна, "Normal" когда потери небольшие (< 10) и "Bad" когда большие

```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame

    float prevState = 0;

    void Update()
    {
        var currentValue = dataSet["Mon_" + i.ToString()];
        if (currentValue >= prevState & statusStart == false & i != dataSet.Count)
        {
            Debug.Log($"Current value: {currentValue}; Prev value: {prevState}");
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
            prevState = currentValue;
            Debug.Log("Good");
        }

        if (currentValue < prevState && prevState - currentValue < 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            Debug.Log($"Current value: {currentValue}; Prev value: {prevState}");
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
            prevState = currentValue;
            Debug.Log("OK");
        }

        if (currentValue < prevState & statusStart == false & i != dataSet.Count)
        {
            Debug.Log($"Current value: {currentValue}; Prev value: {prevState}");
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
            prevState = currentValue;
            Debug.Log("Bad");
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1BV81QVIoKBcMVrGdIcYh_PXkKre1QQJp4glm9SkP4hY/values/%D0%9B%D0%B8%D1%81%D1%821?key=<MY API KEY>");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add($"Mon_{selectRow[0]}", float.Parse(selectRow[1]));
        }
        // foreach (string key in dataSet.Keys)
            // Debug.Log(key);
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

## Выводы

За выполнение я узнал, как пользоваться Google Sheets API и разбирать компоненты игры и их влияние на игровой процесс

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
