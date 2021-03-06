# TrabajoPersonal_AlmudenaManzanal

## Introducción
### Analizando dataset nycflights13:: flights 
Vamos a analizar el siguiente dataset sobre vuelos de US y responder a una serie de preguntas.

```{r}
#install.packages("nycflights13")
flights <- nycflights13::flights
View(flights)
?flights
library(tidyverse)
library(lubridate)
```
#### 1. Encuentra todos los vuelos que llegaron más de una hora tarde de lo previsto. 

```{r}
Retrasos <- flights[flights$arr_delay > 60, ]

head(Retrasos)
```
#### 2. Encuentra todos los vuelos que volaron hacia San Francisco (aeropuertos SFO y OAK) 
```{r}
San_Francisco <- flights[flights$dest == "OAK" | flights$dest == "SFO",]
SF <- filter(flights, dest == "OAK" | dest == "SFO")
head(San_Francisco)
```
#### 3. Encuentra todos los vuelos operados por United American (UA) o por American Airlines (AA) 
```{r}
UnitedAirlines <- filter(flights, carrier == "UA" | carrier == "AA" )
head(UnitedAirlines)
```
#### 4. Encuentra todos los vuelos que salieron los meses de primavera (Abril, Mayo y Junio) 
```{r}
Primavera <- filter(flights, month == c(4, 5, 6))
head(Primavera)
```
#### 5. Encuentra todos los vuelos que llegaron más de una hora tarde pero salieron con menos de una hora de retraso. 
```{r}
DobleRetraso <- filter(flights, dep_delay < 60 & arr_time >60)
head(DobleRetraso)
```
#### 6. Encuentra todos los vuelos que salieron con más de una hora de retraso pero consiguieron llegar con menos de 30 minutos de retraso (el avión aceleró en el aire)
```{r}
RetrasoDep <- filter(flights, dep_delay > 60 & arr_delay < 30)
head(RetrasoDep)
```

#### 7. Encuentra todos los vuelos que salen entre medianoche y las 7 de la mañana (vuelos nocturnos). 
```{r}
NocturnoReal <- filter(flights, dep_time <= 700 | dep_time == 2400)
NocturnoSched <- filter(flights, sched_dep_time <= 700 | sched_dep_time == 2400)
head(NocturnoSched)
```
#### 8. ¿Cuántos vuelos tienen un valor desconocido de dep_time? 
```{r}
summary(is.na(flights$dep_time))
#8255
```
#### 9. ¿Qué variables del dataset contienen valores desconocidos? 
```{r}
summary(is.na(flights))
# Son dep_time, dep_delay, arr_time, arr_delay, tailnum, air_time
```
#### 10. Ordena los vuelos de flights para encontrar los vuelos más retrasados en la salida. ¿Qué vuelos fueron los que salieron los primeros antes de lo previsto? 
```{r}
arrange(flights, dep_delay)
arrange(flights, desc(dep_delay))
```
#### 11. Ordena los vuelos de flights para encontrar los vuelos más rápidos. Usa el concepto de rapidez que consideres. 
```{r}
flights$vel <- flights$distance / flights$air_time
arrange(flights, desc(vel))
```
#### 12. ¿Qué vuelos tienen los trayectos más largos? 
```{r}
  flights %>% 
  group_by(origin, dest) %>% 
  summarise(
      distance = mean(distance)
  ) %>% 
  arrange(desc(distance))
```
#### 13. ¿Qué vuelos tienen los trayectos más cortos? 
```{r}
  flights %>% 
  group_by(origin, dest) %>% 
  summarise(
      distance = mean(distance)
  ) %>% 
  arrange(distance)
```
#### 14. El dataset de vuelos tiene dos variables, dep_time y sched_dep_time muy útiles pero difíciles de usar por cómo vienen dadas al no ser variables continuas. Fíjate que cuando pone 559, se refiere a que el vuelo salió a las 5:59... Convierte este dato en otro más útil que represente el número de minutos que pasan desde media noche
```{r}
datetime <- function(year, month, day, time) 
  {
    make_datetime(year, month, day, time %/% 100, time %% 100)
  }

flights_dt <- 
  flights %>% 
      mutate(
          dep_time = datetime(year, month, day, dep_time),
          sched_dep_time = datetime(year, month, day, sched_dep_time)
             )
flights$sched_dep_time <- hour(flights_dt$sched_dep_time)*60 + minute(flights_dt$sched_dep_time)

flights$dep_time <- hour(flights_dt$dep_time)*60 + minute(flights_dt$dep_time)
```
#### 15. Compara los valores de dep_time, sched_dep_time y dep_delay. ¿Cómo deberían relacionarse estos tres números? Compruébalo y haz las correcciones numéricas que necesitas. 
```{r}
#Se deben restar el tiempo programado al tiempo real de salida y se dan en minutos:
dep <- as.numeric(flights_dt$dep_time - flights_dt$sched_dep_time) / 60
```
#### 16. Investiga si existe algún patrón del número de vuelos que se cancelan cada día. 
```{r}
cancelled_per_day <- 
  flights %>%
  mutate(cancelled = (is.na(arr_delay) | is.na(dep_delay))) %>%
  group_by(year, month, day) %>%
  summarise(
    cancelled_num = sum(cancelled),
    flights_num = n(),
  )


ggplot(cancelled_per_day) +
  geom_point(aes(x = flights_num, y = cancelled_num)) 
```



#### 17. Investiga si la proporción de vuelos cancelados está relacionada con el retraso promedio por día en los vuelos. 
```{r}
cancelled_and_delays <- 
  flights %>%
  mutate(cancelled = (is.na(arr_delay) | is.na(dep_delay))) %>%
  group_by(year, month, day) %>%
  summarise(
    cancelled_prop = mean(cancelled),
    avg_dep_delay = mean(dep_delay, na.rm = TRUE),
    avg_arr_delay = mean(arr_delay, na.rm = TRUE)
  ) %>%
  ungroup()

ggplot(cancelled_and_delays) +
  geom_point(aes(x = avg_dep_delay, y = cancelled_prop))

ggplot(cancelled_and_delays) +
  geom_point(aes(x = avg_arr_delay, y = cancelled_prop))
```


#### 18. Investiga si la proporción de vuelos cancelados está relacionada con el retraso promedio por aeropuerto en los vuelos. 
```{r}
cancelled_and_origin <- 
  flights %>%
  mutate(cancelled = (is.na(arr_delay) | is.na(dep_delay))) %>%
  group_by(origin) %>%
  summarise(
    cancelled_prop = mean(cancelled),
    avg_dep_delay = mean(dep_delay, na.rm = TRUE)
  ) %>%
  ungroup()

cancelled_and_dest <- 
  flights %>%
  mutate(cancelled = (is.na(arr_delay) | is.na(dep_delay))) %>%
  group_by(dest) %>%
  summarise(
    cancelled_prop = mean(cancelled),
    avg_arr_delay = mean(arr_delay, na.rm = TRUE)
  ) %>%
  ungroup()

ggplot(cancelled_and_origin) +
  geom_point(aes(x = avg_dep_delay, y = cancelled_prop))

ggplot(cancelled_and_dest) +
  geom_point(aes(x = avg_arr_delay, y = cancelled_prop))
```



#### 19. ¿Qué compañía aérea sufre los peores retrasos? 
```{r}
delays_carrier <- 
  flights %>%
  group_by(carrier) %>%
  summarise(
    avg_dep_delay = mean(dep_delay, na.rm = TRUE),
    avg_arr_delay = mean(arr_delay, na.rm = TRUE)
  ) %>%
  ungroup()

arrange(delays_carrier, desc(avg_dep_delay))
arrange(delays_carrier, desc(avg_arr_delay))
```



#### 20. Queremos saber qué hora del día nos conviene volar si queremos evitar los retrasos en la salida. 
```{r}
delay_hour <-
flights_dt %>% 
  mutate(hour = hour(dep_time)) %>% 
  group_by(hour) %>% 
  summarise(
    avg_delay = mean(dep_delay, na.rm = TRUE),
    n = n())


  ggplot(delay_hour) +
    geom_line(aes(hour, avg_delay))

```


#### 21. Queremos saber qué día de la semana nos conviene volar si queremos evitar los retrasos en la salida
```{r}
delay_wday <-
flights_dt %>% 
  mutate(wday = wday(sched_dep_time)) %>% 
  group_by(wday) %>% 
  summarise(
    avg_delay = mean(dep_delay, na.rm = TRUE),
    n = n())


  ggplot(delay_wday) +
    geom_line(aes(wday, avg_delay))

```
#### 22. Para cada destino, calcula el total de minutos de retraso acumulado.
```{r}
delay_dest <-
  flights %>% 
  filter(arr_delay > 0) %>% 
  group_by(dest) %>% 
  summarise(
    sum_delay = sum(arr_delay, na.rm = TRUE)
    )
```
#### 23. Para cada uno de ellos, calcula la proporción del total de retraso para dicho destino.
```{r}
delayprop_dest <-
  flights %>%
  filter(arr_delay > 0) %>% 
  group_by(dest) %>% 
  summarise(
    sum_delay = sum(arr_delay, na.rm = TRUE),
    delay_prop = mean(arr_delay, na.rm =TRUE)
    )
```

#### 24. Es hora de aplicar todo lo que hemos aprendido para visualizar mejor los tiempos de salida para vuelos cancelados vs los no cancelados. Recuerda bien qué tipo de dato tenemos en cada caso. ¿Qué deduces acerca de los retrasos según la hora del día a la que está programada el vuelo de salida?
```{r}
flights_dt$cancelled <-  ifelse((is.na(flights_dt$arr_delay) | is.na(flights_dt$dep_delay)) == TRUE, print(TRUE), print(FALSE)) 

cancelled <- flights_dt %>% 
  filter(cancelled == TRUE)

not_cancelled <- flights_dt %>% 
  filter(!is.na(dep_delay), !is.na(arr_delay))

sched_dep2 <- flights_dt %>% 
  mutate(hour = hour(sched_dep_time)) %>% 
  group_by(hour) %>% 
  summarise(
    ave_delay = mean(dep_delay, na.rm = TRUE),
    n = n()
  )

ggplot(sched_dep2, aes(hour, ave_delay)) +
  geom_line()

#Muchos mas retrasos a ultimas horas del dia.

```

#### 25. Subir la carpeta a github y facilitar la url.
```{r}
# https://github.com/amanzgdl/TrabajoPersonal_AlmudenaManzanal.git 
```

#### 26. Al finalizar el documento agrega el comando sessionInfo()
```{r}
sessionInfo()
```
