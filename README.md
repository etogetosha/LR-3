# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:
- Ясаков Антон Павлович
- НМТ-213929
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### В данной лабораторной работе мы создадим ML-агент и будем тренировать нейросеть, задача которой будет заключаться в управлении шаром. Задача шара заключается в том, чтобы оставаясь на плоскости находить кубик, смещающийся в заданном случайном диапазоне координат.

-	Создим новый пустой 3D проект на Unity.
![image](https://user-images.githubusercontent.com/114253132/200176202-f83faada-a65b-473e-b8bc-fa4084c872ef.png)
-	Скачаем папку с ML агентом. 

![1](https://user-images.githubusercontent.com/114507692/198083887-05e6c871-dfb6-4ecf-941a-31ec8bcfd0c4.png)

-	В созданный проект добавляем ML Agent, выбрав Window - Package Manager - Add Package from disk. Последовательно добавляем .json – файлы:
	ml-agents-release_19 / com,unity.ml-agents / package.json
	ml-agents-release_19 / com,unity.ml-agents.extensions / package.json
- Основные действия в Anaconda Prompt

![image](https://user-images.githubusercontent.com/90757310/197793868-91705f59-0d51-4245-949a-d475069483e5.png)

-	Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:

![image](https://user-images.githubusercontent.com/90757310/197794052-44ac3201-b68b-403e-bfd7-98bcb5d13877.png)

![image](https://user-images.githubusercontent.com/90757310/197794150-ee542e25-fbd2-41c4-80b1-a1f1d182101c.png)

-	Создадим на сцене плоскость, куб и сферу, а затем простой C# скрипт и подключим его к сфере:

![image](https://user-images.githubusercontent.com/114253132/200180698-e5cc6cf7-7a22-484f-9387-43c5caf32210.png)

-	В скрипт-файл RollerAgent.cs добавляем код:


```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8 - 4, 0.5f, Random.value * 8 - 4);

    }

    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}



```


-	В корень проекта добавим файл конфигурации нейронной сети.

![3](https://user-images.githubusercontent.com/114507692/198088717-203088bc-fe84-4288-b7f4-a9d11dcc7656.png)

- Запустим ml-агента

-	Сделаем 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запустим симуляцию сцены и выполним проверку.

![image](https://user-images.githubusercontent.com/114253132/200181377-0ca1c266-bd63-47eb-b090-0bb707fbdc7f.png)

![image](https://user-images.githubusercontent.com/114253132/200181382-5e5af10c-5b55-4e68-b8c8-20c4240bd342.png)

![image](https://user-images.githubusercontent.com/114253132/200181388-a8292c1c-02b0-4c5f-b869-7cd202d909ed.png)

![image](https://user-images.githubusercontent.com/114253132/200181395-4ea748fb-1d40-4abc-b19e-5b39406196a7.png)

![image](https://user-images.githubusercontent.com/114253132/200181397-a78929ae-cfc7-4cac-9046-a1b4c3a43549.png)

![image](https://user-images.githubusercontent.com/114253132/200181459-39290f6f-705f-44ca-b2c6-79c327e7b359.png)

![image](https://user-images.githubusercontent.com/114253132/200181465-b069e9aa-5adf-4f7b-8b82-f2d26b1b0442.png)

![image](https://user-images.githubusercontent.com/114253132/200181471-296d65ae-9119-4c33-8ad0-19edd0388b31.png)

![image](https://user-images.githubusercontent.com/114253132/200181481-75fd8bbc-27f5-4af5-addc-cbf97b5fd778.png)

![image](https://user-images.githubusercontent.com/114253132/200181487-c2e1302f-d501-4531-81d1-817f6dc7527b.png)

## Выводы
В ходе лабораторной работы я обучил ML-агента для реализации машинного обучения в среде Unity. Для обучения я использовал библиотеки mlagents 0.28.0 и torch версии 1.7.1
