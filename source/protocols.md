# Protocols

## Energy usage

All values approximate

|Activity|W|A|
|----------------------|------|------|
|Idle, both PCs running|~140|~5.2|
|all motors ready (set to freeze), no z-lift|~165|~6.8|
|Z-lift ready|~170|~7|
|Z-lift going up|~220|~9|
|Omnibase |
|idle | ~160 | |
|driving steady | ~230 | |
|accelerating | up to ~320 | |
|driving and turning | up to ~320 | |
|turning steady | ~220 | |
|turning accelerating | up to ~280 | |

## Tested kernels w RTAI

|Kernel Vers        | Source  |Status   |Package | Comment |
|-------------------|---------|---------|--------|---------|
| 3.8.13-rtai       | ahoarau | freeze |  |  |
| 3.8.13 + rtai rc3 | smeyerz | freeze  |  | |
| 3.8.13 + rtai rc4 | smeyerz | freeze  |  | |
| 3.10.32-rt        | ahoarau | works w/o wlan stick  | linux-image-3.10.32-rt_3.10.32-rt-10.00.Custom_amd64.deb | WITHX86TIMER |
| 3.10.32-rtai4.0   | ahoarau | kernel panic |  | |
| 3.10.32-rtwar3    | ahoarau | kernel panic |  | |
| 3.10.32-rtwar4    | ahoarau | kernel panic |  | |
| 3.10.44 | shabbyx + smeyerz | freeze | | |
| 3.14.17 | ? | missing stuff | | |
| 3.16.7 | ? | no real time | | |
