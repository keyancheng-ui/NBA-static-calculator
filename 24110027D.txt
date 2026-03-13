#!/bin/bash

# check the number of parameters
if [ $# -lt 2 ]; then
    echo "You need to input more information."
    exit 1
fi

PLAYER="$1" # the first parameter
shift # remove the first parameter
STATS=("$@") # set an array for other parameters

shopt -s nullglob
NBA_FILES=(nba*.txt)
shopt -u nullglob

# execute the command and take the output as the parameter value
if [ -z "$NBA_FILES" ]; then
    echo "No data file found."
    exit 1
fi

# define a function to retrieve statistics
retrieve(){
    # statistics
    local GAMES=0 MINUTES=0 FGM=0 FGA=0 TPM=0 TPA=0 FTM=0 FTA=0
    local OREB=0 DREB=0 REB=0 AST=0 STL=0 BLK=0 TO=0 PF=0 PTS=0 PLUS_MINUS=0
    
    local player_name="$1"
    shift
    local -a req_stats=("$@")

    # retrieve data for a certain player
    for FILE in "${NBA_FILES[@]}"; do
        PLAYER_DATA=$(awk -v player="$player_name" '
        BEGIN { found=0 }

        # jump lines that contain matching contents
        /PLAYER/ { next }
        /TOTALS/ { next }
        /DNP/ { next }

        # match players
        $0 ~ player {
            found = 1
            
            split($(NF-19), time, ":")
            if (length(time) == 2) {
                minutes = time[1] + time[2]/60
            } else {
                minutes = 0
            }
            
            # key data (NF represents total word number in this line)
            fgm = $(NF-18) + 0
            fga = $(NF-17) + 0
            tpm = $(NF-15) + 0
            tpa = $(NF-14) + 0
            ftm = $(NF-12) + 0
            fta = $(NF-11) + 0
            oreb = $(NF-9) + 0
            dreb = $(NF-8) + 0
            reb = $(NF-7) + 0
            ast = $(NF-6) + 0
            stl = $(NF-5) + 0
            blk = $(NF-4) + 0
            to = $(NF-3) + 0
            pf = $(NF-2) + 0
            pts = $(NF-1) + 0
            plus_minus = $NF + 0

            print minutes "," fgm "," fga "," tpm "," tpa "," ftm "," fta "," oreb "," dreb "," reb "," ast "," stl "," blk "," to "," pf "," pts "," plus_minus
        }
        
        END { if (!found) print "No such player."}
        ' "$FILE")

        if [ "$PLAYER_DATA" != "No such player." ]&&[ -n "$PLAYER_DATA" ]; then
            # add data together
            GAMES=$((GAMES + 1))
            
            while IFS=',' read -r minutes fgm fga tpm tpa ftm fta oreb dreb reb ast stl blk to pf pts plus_minus; do
                MINUTES=$(awk "BEGIN {printf \"%.1f\", $MINUTES + $minutes}")
                # a format to handle floating numbers
                FGM=$((FGM + fgm))
                FGA=$((FGA + fga))
                TPM=$((TPM + tpm))
                TPA=$((TPA + tpa))
                FTM=$((FTM + ftm))
                FTA=$((FTA + fta))
                OREB=$((OREB + oreb))
                DREB=$((DREB + dreb))
                REB=$((REB + reb))
                AST=$((AST + ast))
                STL=$((STL + stl))
                BLK=$((BLK + blk))
                TO=$((TO + to))
                PF=$((PF + pf))
                PTS=$((PTS + pts))
                PLUS_MINUS=$((PLUS_MINUS + plus_minus))
            done <<< "$PLAYER_DATA"
        fi
    done

    # calculate the average and the percent
    if [ $GAMES -gt 0 ]; then
        AVG_MINUTES=$(awk "BEGIN {printf \"%.1f\", $MINUTES / ($GAMES)}")
        AVG_PTS=$(awk "BEGIN {printf \"%.1f\", $PTS / $GAMES}")
        AVG_OREB=$(awk "BEGIN {printf \"%.1f\", $OREB / $GAMES}")
        AVG_DREB=$(awk "BEGIN {printf \"%.1f\", $DREB / $GAMES}")
        AVG_REB=$(awk "BEGIN {printf \"%.1f\", $REB / $GAMES}")
        AVG_AST=$(awk "BEGIN {printf \"%.1f\", $AST / $GAMES}")
        AVG_STL=$(awk "BEGIN {printf \"%.1f\", $STL / $GAMES}")
        AVG_BLK=$(awk "BEGIN {printf \"%.1f\", $BLK / $GAMES}")
        AVG_TO=$(awk "BEGIN {printf \"%.1f\", $TO / $GAMES}")
        AVG_PF=$(awk "BEGIN {printf \"%.1f\", $PF / $GAMES}")
        AVG_PLUS_MINUS=$(awk "BEGIN {printf \"%.1f\", $PLUS_MINUS / $GAMES}")

        if [ $FGA -gt 0 ]; then
            FGP=$(awk "BEGIN {printf \"%.1f\", $FGM * 100 / $FGA}")
        else
            FGP=0
        fi

        if [ $TPA -gt 0 ]; then
            TPP=$(awk "BEGIN {printf \"%.1f\", $TPM * 100 / $TPA}")
        else
            TPP=0
        fi

        if [ $FTA -gt 0 ]; then
            FTP=$(awk "BEGIN {printf \"%.1f\", $FTM * 100 / $FTA}")
        else
            FTP=0
        fi
    else
        echo "No such player found."
        exit 1
    fi
   
    # the output
    for STAT in "${req_stats[@]}"; do
        case $STAT in
            "MIN")
                echo "MIN $AVG_MINUTES"
                ;;
            "PTS")
                echo "PTS $AVG_PTS"
                ;;
            "OREB")
                echo "OREB $AVG_OREB"
                ;;
            "DREB")
                echo "DREB $AVG_DREB"
                ;;
            "REB")
                echo "REB $AVG_REB"
                ;;
            "AST")
                echo "AST $AVG_AST"
                ;;
            "STL")
                echo "STL $AVG_STL"
                ;;
            "BLK")
                echo "BLK $AVG_BLK"
                ;;
            "TO")
                echo "TO $AVG_TO"
                ;;
            "PF")
                echo "PF $AVG_PF"
                ;;
            "FGM")
                AVG_FGM=$(awk "BEGIN {printf \"%.1f\", $FGM / $GAMES}");
                echo "FGM $AVG_FGM"
                ;;
            "FGA")
                AVG_FGA=$(awk "BEGIN {printf \"%.1f\", $FGA / $GAMES}");
                echo "FGA $AVG_FGA"
                ;;
            "FG%")
                echo "FG% $FGP"
                ;;
            "3PM")
                AVG_TPM=$(awk "BEGIN {printf \"%.1f\", $TPM / $GAMES}");
                echo "3PM $AVG_TPM"
                ;;
            "3PA")
                AVG_TPA=$(awk "BEGIN {printf \"%.1f\", $TPA / $GAMES}");
                echo "3PA $AVG_TPA"
                ;;
            "3P%")
                echo "3P% $TPP"
                ;;
            "FTM")
                AVG_FTM=$(awk "BEGIN {printf \"%.1f\", $FTM / $GAMES}");
                echo "FTM $AVG_FTM"
                ;;
            "FTA")
                AVG_FTA=$(awk "BEGIN {printf \"%.1f\", $FTA / $GAMES}");
                echo "FTA $AVG_FTA"
                ;;
            "FT%")
                echo "FT% $FTP"
                ;;
            "+/-")
                echo "+/- $AVG_PLUS_MINUS"
                ;;
            *)
                echo "No such stat: $STAT"
                ;;
        esac
    done
}

if [[ "$PLAYER" == *" "* ]]; then
    retrieve "$PLAYER" "${STATS[@]}"
else
    unset players
    declare -A players
    for FILE in "${NBA_FILES[@]}"; do
        while IFS= read -r fullname; do
            players["$fullname"]=1
        done < <(
            awk -v player="$PLAYER" '
            BEGIN { IGNORECASE = 1 }
            
            /PLAYER/ { next }
            /TOTALS/ { next }
            /DNP/ { next }
 
            $0 ~ player {
                arr["C"]; arr["G"]; arr["F"]
                var=$(NF-20)
                if(var in arr){
                    fullname=$1
                    for(i = 2; i < NF-20; i++){
                        fullname=fullname " " $i
                    }
                    print fullname
                }
                else{
                    fullname=$1
                    for(i = 2; i < NF-19; i++){
                        fullname=fullname " " $i
                    }
                    print fullname
                }      
            }
            ' "$FILE"
        )
    done
    player_names=("${!players[@]}")

    for Player in "${player_names[@]}"; do
        echo $Player
        retrieve "$Player" "${STATS[@]}"
    done
fi
