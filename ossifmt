#!/bin/sh

USAGE="ossifmt [fmtr] [files...]

FORMATTERS:
alarm		hardware alarm log
block		measurements blockage form
first string...	dump the raw output from that command
hist		history
hwerr		hardware error log
hwerri		hardware error log incremental
mst		MST buffer
prep		preprocessed output (one record per line)		
raw		each field separated by a space
unity sep	each field spearated by a "sep"	
"

if [ $# -lt 2 ]
then
	echo "${USAGE}"
	exit 1
fi

TMPAWK=/tmp/ofawk$$
TMPIN=/tmp/of$$
TMPF=/tmp/off$$
TMPU=/tmp/ofu$$
TMPS="$TMPAWK $TMPIN $TMPF $TMPU"

trap 'rm -f $TMPS; exit 255' 1 2 15

[ -x /usr/bin/gawk ] && alias gawk=/usr/bin/gawk

dprog()	# command-to-display formatting-program
{
local CMD="$1" # define function-local variables
local FMT="$2"
shift 2

cat >$TMPAWK <<!
	BEGIN { FS="\t" }

	(\$1 == "c" && \$2 ~ /^$CMD/) {
		START = 1
		if (MARK == 2) print
		next
	}

	(START == 0) { next }

	(\$1 == "t") {
		if (MARK == 1) print "end-of-dump"
		else if (MARK == 2) print
		START = 0
		next
	}

	(\$1 == "d") {
		$FMT
		print B
	}
!
gawk -f $TMPAWK "$@" -
}

unity()
{
	dprog '.' 'B = substr($0,3); gsub(/	/,UFS,B)' UFS="${1-@}" MARK="${2-2}"
}

alarm()
{
	dprog 'di.*ala' '
	{
		B = sprintf("%-7s@%-8s@%1.1s@%-12.12s@%-7s@",$2,$3,$4,$5,$6)
		B = B sprintf("%5s@%1s@%1s@%02d/%02d@%02d:",$7,$8,$9,$10,$11,$12)
		B = B sprintf("%02d@%02d/%02d@%02d:%02d",$13,$14,$15,$16,$17)

		if (MARK)	gsub(/ /,"",B)
		else		gsub(/@/," ",B)
	}
	' $@
}

hwerr()
{
	dprog 'di.*err' '
	{
		pname = $3
		gsub(/ /,"-",pname)
		if (pname == "") pname = "."
		aname = $5 
		if (aname == "") aname = "."
		B = sprintf("%-7s@%-8s@%-7.7s@%5d@%5d@",\
			pname,$4,aname,$6,$7)

		if ($2 ~ /^HIG/ || $2 ~ /RESO/)
		{
			B = B sprintf("%02d%02d:%02d%02d",$8,$9,$10,$11)
			if (!MARK) B = B sprintf(".%02d/%02d",$12,$13)
			B = B sprintf("@%02d%02d:%02d%02d",$14,$15,$16,$17)
			if (!MARK) B = B sprintf(".%02d/%02d",$18,$19)
			B = B sprintf("@%3d",$20)
		}
		else
		{
			B = B sprintf("%02d%02d:%02d%02d",$8,$9,$10,$11)
			B = B sprintf("@%02d%02d:%02d%02d",$12,$13,$14,$15)
			cntamt = sprintf("(%d/%d)",$6/256+1,$6%256)
			B = B sprintf("@%3d@%8s",$16,cntamt)
		}

		if (MARK)	gsub(/ /,"",B)
		else		gsub(/@/," ",B)
	}
	' $@
}

hwerri()
{
	hwerr MARK=1 |
	gawk '
	BEGIN { FS="@" }

	($1 == "end-of-dump") { DUMP++; continue }

	{
		# TAG=FTAG=sprintf("%-8s %5d %-7s",$2,$4,$1)
		TAG=FTAG=sprintf("%-8s %5d %-7s",$2,$4,$5)

		if (DUMP > 0)
		{
			# number of new errors of FTAG type
			new = $8 - CNT[FTAG]

			if (new < 0)
			{
				# count was cleared and restarted.  Know that
				# at least as many as there are now (possibly
				# more)
				new = $8
			}
			else if (new == 0 && CNT[FTAG] == 255 && LDT[FTAG] != $7)
			{
				# count has pegged at limit.  if new last date
				# then we can assume that at least one has
				# occurred
				new = 1
			}
			TOT[TAG] += new
		}

		CNT[FTAG] = $8
		LDT[FTAG] = $7
	}

	END {
		for (x in TOT)
		{
			if (TOT[x] > 0)
				printf "%4d  %s\n",TOT[x],x
		}
	}
	' - |
	sort +1
}

hist()
{
#	echo " Date  Time Port      Login   Actn  Object       Qualifier"
#	echo

	dprog 'li.*hist' 'B=sprintf("%5s %5s %-9s %-7s %-5s %-12s %-20s",
		$3,$4,$5,$6,$7,$8,$9)'
}

mblock()
{
	dprog 'list.*block' '
	{
		B = sprintf("%2d %-4.4s TDM:(%5d(%2d%%)u",\
						$4,$5,$6,(100*$6)/17352)
		B = B sprintf("%5dp %5db) FIB:(%3dts %5d(%2d%%)u %5dp",\
					$7,$8,$9,$10,(100*$10)/($9*36*2),$11)
		B = B sprintf("%5db)",$12)
	}
	'
}

first()
{
gawk '
($0 ~ /^c.*'"$*"'/) { START = 1 }
(START == 1) {
	print
	if ( $1 == "t") exit 0
}
' -
}

mst()
{
	gawk '
	($0 ~ /^d[ 	][0-9]/) {
		if (LINE) {
			print LINE
		}
		LINE=substr($0,2)
#		continue
	}

	($1 == "d") {
		LINE = LINE " " substr($0,2)
		#continue
	}

	END {
		print LINE
	}
	' - |
	sed -e 's/[ 	][ 	]*/ /g;s,\(/[0-9][0-9]\) ,\1-,'
}


if [ $# = 0 ]
then
	CMD=raw
else
	CMD=$1
	shift
fi

FILES=
ARGS=
for x
do
	case "$1" in
	-*)	ARGS="$ARGS $x";;
	*)	if [ -f "$x" ]
		then
			FILES="$FILES $x"
		else
			ARGS="$ARGS $x"
		fi
		;;
	esac
done
case "$CMD" in
first)		first $ARGS;exit 0;;
esac

gawk '
# The tilde (~) operator allows you to test a regular expression against a
# field. Here $1 is first field
($1 ~ /^[cfdnt]/) {
# get first character of first word
	TYPE = substr($1,1,1)
	if (TYPE == LAST_TYPE)
	{
		NFIELDS += NF
		if (NFIELDS < 100)
		{
			LINE = LINE "	" substr($0,2)
		}
	}
	else
	{
		if (LINE) print LINE
		if (TYPE == "n")
		{
			LINE=""
			NFIELDS = 0
		}
		else
		{
			LINE= TYPE "	" substr($0,2)
			NFIELDS = NF + 1
		}
	}
	LAST_TYPE = TYPE
}
' $FILES |
case "$CMD" in
ala*)		alarm;;
bl*)		mblock;;
hist)		hist;;
hwerr)		hwerr;;
hwerri)		hwerri;;
mst)		mst;;
raw)		unity " ";;
unity)		unity "$ARGS";;
prep)		cat;;
*)		echo "${USAGE}"
		exit 1;;
esac

#rm -f $TMPS
