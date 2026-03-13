# NBA static calculator

## a calculator to analyze NBA players' performances

If you like to watch NBA games, and you want to see how a specific player performs over multiple matches (*that is to say, his average statistic*), this program will help you.

The designed bash script can read all NBA data files under the same document in the format of **nbaYYMMDDteam1team2.txt**, using a special bash program named **awk**.

When users execute this script, they need to input the player they want to assess, the statistic they want to see (*for example, PTS for points, REB for rebounds, etc.*), then the program will go through all this player's match data within your provided games, calculate the values, and show you an average statistic list.

In addition, this program can help you compare multiple players' performances if they have the same names (*for example, LeBron James and James Harden*). Just input their shared names and this program will print out their statistics separately.