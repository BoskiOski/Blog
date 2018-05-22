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

Metoda timeOnTargetByCountryin71Aggregatee sumuje czas jaki został poświęcony przez poszczególne kraje na operacje bombowe podczas roku 1971.  Te informacje zostają następnie zapisane w pliku txt.
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

| Kraj | Laczny czas nalotow |
| --- | --- |
| AUSTRALIA | 14756.0 |
| LAOS | 46147.0 |
| UNITED STATES OF AMERICA | 2204323.0 |
| VIETNAM (SOUTH) | 483155.0 |
