# TUM-ParProg-Final

Performance Analysis with `perf` and `flamegraph`

## Command Example

```shell
# Declare the program and matrix files
PROG=hybridge
FILES="size2048x2048 size1024x1024 size512x512"

# On-CPU Time
for f in $FILES
do
  sudo perf record -g mpirun --allow-run-as-root -n 3 ./${PROG} ./ge_data/${f}
  sudo perf script -i perf.data &> perf.unfold
  ./stackcollapse-perf.pl perf.unfold &> perf.folded
  grep ${PROG} perf.folded \
    | ./flamegraph.pl \
    --title="On-CPU Time Flame Graph: $ ./${PROG} ./ge_data/${f} (bonus)" \
    > oncpu.bonus.${f}.svg
done
  
# Off-CPU Time
./${PROG} ./ge_data/size8192x8192 &
sudo offcputime-bpfcc -df -p `pgrep -nx ${PROG}` 30 > offcpu.stacks
pkill ${PROG}
./flamegraph.pl --color=io \
    --title="Off-CPU Time Flame Graph (30s): $ ./${PROG} ./ge_data/size8192x8192 (bonus)" \
    --countname=us < ./offcpu.stacks > offcpu.bonus.size8192x8192.svg
```

## Reference
* [Linux perf Examples](http://www.brendangregg.com/perf.html)
* [CPU Flame Graphs](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html)
* [Off-CPU Analysis](http://www.brendangregg.com/offcpuanalysis.html)

