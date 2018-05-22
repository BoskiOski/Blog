## Projekt na egzamin z Technologi NoSQL
### Agregacje korzystające z Aggregation Pipeline i Map-Reduce
> Mateusz Breza 224612, Oskar Marcinkiewicz 224622, Bartosz Wiśniewski 224617

Do projektu została użyta baza danych misji bombowych podczas wojny w Wietnamie. Zawiera 11969 rekordów.

## Docker

Do poprawnego działania projektu wymagana jest instalacja : Docker, Docker compose

Aplikacja została napisana w języku python, używaną bazą danych jest mongo. Znajdują się one w osobnych kontenerach. <br />  Wszystkie komendy należy wykonać w folderze pythondocker.


1.Wpisujemy "sudo docker-compose build" aby zbudować program. 

2.Wpisujemy "sudo docker-compose up" aby uruchomić program.

W tym momencie aplikacja uruchamia się i jest dostępna na porcie 5000 local hosta

## Aplikacja

Metoda timeOnTargetByCountryin71Aggregatee sumuje czas jaki został poświęcony przez poszczególne kraje na operacje bombowe podczas roku 1971 z użyciem aggregation pipeline.  Te informacje zostają następnie zapisane w pliku txt.
```python
def timeOnTargetByCountryin71Aggregatee():
    pipeline = [
        {"$project":{"COUNTRYFLYINGMISSION" : 1 , "TIMEONTARGET" : 1, "year" : {"$year" :'$MSNDATE'}}},
        {"$match":{"year": 1971 }},
        {"$group": {"_id": "$COUNTRYFLYINGMISSION", "count": {"$sum": "$TIMEONTARGET"}}}
    ]

    result = db.thor.aggregate(pipeline)
    ret = "<h1>Time on target for each country in 1971:</h1>" 
    for el in result:
        if(el['_id'] == ''):
           continue
        ret = ret + "<p>"+ str(el['_id']) + ": "
        ret = ret + str(el['count']) + "</p>" 
    
    return ret
```

| Kraj | Laczny czas nalotow |
| --- | --- |
| AUSTRALIA | 14756.0 |
| LAOS | 46147.0 |
| UNITED STATES OF AMERICA | 2204323.0 |
| VIETNAM (SOUTH) | 483155.0 |

Metoda usaBombingByMonthAggregate analizuje ilość ataków jakie zostały przeprowadzone w danym miesiącu przez usa podczas całego okresu wojny. Te informacje zostają następnie wyświetlone na stronie. Metoda używa aggregation pipeline.
```python
def usaBombingByMonthAggregate():

    pipeline = [
        {"$match": {"COUNTRYFLYINGMISSION": "UNITED STATES OF AMERICA"}},
        {"$group": {"_id": {"$month":"$MSNDATE"}, "count" : {"$sum" : 1}}}
    ]
    result = db.thor.aggregate(pipeline)
    month_lst = ['January', 'February', 'March', 'April', 'May', 'June', 'July',
              'August', 'September', 'October', 'November', 'December']
    ret = "<h1>Number of USA attacks in each month:</h1>"
    for el in result:
        ret = ret + "<p>"+ str(month_lst[int(el['_id']-1)]) + ": "
        ret = ret + str(el['count']) + "</p>"
    return ret
```

Metoda usaBombingByMonthMapReduce również zlicza amerykańskie naloty bombowe, po czym tworzy wykres obrazujący przetworzone dane.
```python
def usaBombingByMonthMapReduce():
    map = Code("function () {"
            "emit(this.MSNDATE.getMonth(), 1);"
            "}")

    reduce = Code("function(keyCountry, values) { "
               " return Array.sum(values); "
               " };")

    result = db.thor.map_reduce(map, reduce, "firstMR", query = {"COUNTRYFLYINGMISSION":"UNITED STATES OF AMERICA"})

    month_lst = ['January', 'February', 'March', 'April', 'May', 'June', 'July',
              'August', 'September', 'October', 'November', 'December']
    xList = []
    yList = []
    for doc in result.find():
        if(doc['_id'] != ''):
            xList.append(month_lst[int(doc['_id'])])
            yList.append(doc['value'])

    print xList
    print yList

    trace = go.Bar(x=xList, y= yList)
    data = [trace]
    layout = go.Layout(title='USA Bombing by Month', width=800, height=640)
    fig = go.Figure(data=data, layout=layout)
    py.image.save_as(fig, filename='static/images/usaBombingByMonthMapReduce.png')

    full_filename = os.path.join(app.config['UPLOAD_FOLDER'], 'usaBombingByMonthMapReduce.png')
    return render_template("wykres.html", user_image = full_filename)
```
Utworzony [Wykres](https://github.com/BoskiOski/Blog/blob/gh-pages/zdjecie.png).
