

** Run CODENN **

Requirements

> Torch (http://torch.ch/docs/getting-started.html)
> fblualib (https://github.com/facebook/fblualib)
> antlr4 for parsing C# (pip install antlr4-python2-runtime)

Setup environment

> export PYTHONPATH=~/codenn/src/:~/codenn/src/sqlparse
> export CODENN_DIR=~/codenn/

Build the datasets

> cd src/model
> th buildData.lua -language csharp -working_dir ./csharp_data/
> th buildData.lua -language sql -working_dir ./sql_data/


For Csharp

> th buildDataPy.lua -language csharp -working_dir ./c_t_2_2/ 
> th main.lua -bleufile ~/nlidb/data/stackoverflow/csharp/dev/ref.txt -gpuidx 2 -rnn_size 400 -working_dir ./c_t_2_2/
> cp rnn2.model1 bestc1
> cp rnn2.model2 bestc2

Generate predictions on dev and test

> th predict.lua  -model1 bestc1 -model2 bestc2 -working_dir ./c_t_2_2/ -testfile human.data -beamsize 10 -gpuidx 3  
> mv /tmp/lua_4cUZqd  ~/nlidb/data/stackoverflow/csharp/dev/lstm.txt

> th buildDataPy.lua -language csharp -working_dir ./c_t_2_2/ -devfile ~/nlidb/data/stackoverflow/csharp/test/data.txt
> th predict.lua  -model1 bestc1 -model2 bestc2 -working_dir ./c_t_2_2/ -testfile devfile.data -beamsize 10 -gpuidx 3  
> mv /tmp/lua_EFYlWz   ~/nlidb/data/stackoverflow/csharp/test/lstm.txt


Generate Metrics

> python ../utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/csharp/test/lstm.txt
> python ../utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/csharp/dev/lstm.txt
> python ../utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/csharp/test/lstm.txt
> python ../utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/csharp/dev/lstm.txt


For SQL

> th buildDataPy.lua -language sql -working_dir ./s_t_3_2_rse/
> th main.lua -bleufile ~/nlidb/data/stackoverflow/sql/dev/ref.txt -gpuidx 1 -rnn_size 400 -working_dir ./s_t_3_2_rse/    
> cp rnn1.model1 bestsql1
> cp rnn1.model2 bestsql2

Generate predictions on dev and test

> th predict.lua  -model1 bestsql1 -model2 bestsql2 -working_dir ./s_t_3_2_rse/ -testfile human.data -beamsize 10 -gpuidx 3  
> mv /tmp/lua_EFYlWz   ~/nlidb/data/stackoverflow/sql/dev/lstm.txt

> th buildDataPy.lua -language sql -working_dir ./s_t_3_2_rse/ -devfile ~/nlidb/data/stackoverflow/sql/test/data.txt
> th predict.lua  -model1 bestsql1 -model2 bestsql2 -working_dir ./s_t_3_2_rse/ -testfile devfile.data -beamsize 10 -gpuidx 3  
> mv /tmp/lua_EFYlWz   ~/nlidb/data/stackoverflow/sql/test/lstm.txt

Generate Metrics

> python ../utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/sql/dev/lstm.txt
> python ../utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/sql/test/lstm.txt
> python ../utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/sql/dev/lstm.txt
> python ../utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 < ~/nlidb/data/stackoverflow/sql/test/lstm.txt


** Run IR **

This script works for csharp and sql for dev and test

> python nearestNeighbor.py
> cat result* > ~/nlidb/data/stackoverflow/sql/dev/ir.txt


> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/ir.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/ir.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/ir.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/ir.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/ir.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/ir.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/ir.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/ir.txt


** Run MOSES **

>./moses_train.sh

Im having a bug with kenlm. So I had to train it on panoramix.

> kenlm -o 3 -S 80% -T /tmp < tmp_stackoverflow_training.clean.nl > lm/tmp_stackoverflow_training.arpa
> scp tmp_stackoverflow_training.clean.nl sviyer@panoramix:/tmp/sql.txt
> scp sviyer@panoramix:/tmp/sql.arpa lm/tmp_stackoverflow_training.arpa
> arpa2bin lm/tmp_stackoverflow_training.arpa lm/tmp_stackoverflow_training.blm

Launch training

> tmoses -root-dir . -corpus tmp_stackoverflow_training.clean  -f sql -e nl -alignment grow-diag-final-and -reordering msd-bidirectional-fe -lm 0:3:`pwd`/lm/tmp_stackoverflow_training.blm -cores 60 -external-bin-dir=/home/sviyer/training-tools/ -mgiza -mgiza-cpus 8 -score-options '--NoLex --OnlyDirect'

Run MERT

> mert tmp_stackoverflow_validation.clean.sql  moses/tmp_stackoverflow_validation.nl ${MOSES_DIR}/bin/moses model/moses.ini   --mertdir ${MOSES_DIR}/bin --decoder-flags="-threads 64" --batch-mira --return-best-dev --batch-mira-args '-C 0.0001'

Run Testing with the mert tuned weights

>./moses_test.sh

Compute Metrics for MOSES

> python ~/nlidb/src/core/utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/moses.txt

> python ~/nlidb/src/core/utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/moses.txt

** Run RUSH **

For SQL

> bash srini_train.sh --trainfile ~/nlidb/data/stackoverflow/sql/train_typed.txt --validfile ~/nlidb/data/stackoverflow/sql/valid_typed.txt --devfile ~/nlidb/data/stackoverflow/sql/dev/data_typed.txt  --workdir ./working_dir 

This creates rush_c_300.txt

> mv rush_c_300.txt ~/nlidb/data/stackoverflow/sql/dev/rush.txt

> th summary/run.lua  -modelFilename working_dir/models/model.th  -inputf ~/nlidb/data/stackoverflow/sql/test/data_typed.txt -length 10 -blockRepeatWords > ~/nlidb/data/stackoverflow/sql/test/rush.txt

Compute metrics

> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/rush.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/rush.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/dev/rush.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/sql/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/sql/test/rush.txt

For CSharp

set LR for C# = 0.01

> bash srini_train.sh -t ~/nlidb/data/stackoverflow/csharp/train_typed.txt -v ~/nlidb/data/stackoverflow/csharp/valid_typed.txt -d ~/nlidb/data/stackoverflow/csharp/dev/data_typed.txt -w ./working_csharp/ 

> mv rush_c_300.txt ~/nlidb/data/stackoverflow/csharp/dev/rush.txt

> th summary/run.lua  -modelFilename working_csharp/models/model.th  -inputf ~/nlidb/data/stackoverflow/csharp/test/data_typed.txt -length 10 -blockRepeatWords > ~/nlidb/data/stackoverflow/csharp/test/rush.txt

> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/rush.txt
> python utils/getMetrics.py --metric meteor --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/rush.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/test/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/test/rush.txt
> python utils/getMetrics.py --metric bleu --bleufile /home/sviyer/nlidb/data/stackoverflow/csharp/dev/ref.txt --numref 3 <  ~/nlidb/data/stackoverflow/csharp/dev/rush.txt

** Naturalness **

> { awk -F'\t' 'BEGIN {OFS="\t"}; {print "SQL","IR",$1,$2}' ir.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "SQL","MOSES",$1,$2}' moses.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "SQL","LSTM",$1,$2}' lstm.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "SQL","RUSH",$1,$2}' rush.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "C#","RUSH",$1,$2}' ../../csharp/test/rush.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "C#","LSTM",$1,$2}' ../../csharp/test/lstm.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "C#","MOSES",$1,$2}' ../../csharp/test/moses.txt ; awk -F'\t' 'BEGIN {OFS="\t"}; {print "C#","IR",$1,$2}' ../../csharp/test/ir.txt ; } | shuf | curl -F 'f:1=<-' ix.io

** Informativeness **

> cd /home/sviyer/nlidb/data/stackoverflow/sql
> python informativeness.py

** Mean Reciprocal Rank **

This command makes a json file which contains the mrr schedule.

Format of the schedule json file is

> [ run1, run2, ... , run20     ]
> Each run is [ mrr1, mrr2, ..., mrr100        ]
> Each mrr is { id: X, nl: Y, candidates: [ list of 50 ids ]  }

Parameters for C# and SQL and Miltos are in the file itself

** CODENN **

SQL MRR 

> python prepare_mrr_data.py # We need to make this for dev and test

> th buildDataPy.lua -language sql -working_dir ./s_t_3_2_rse/  -devfile ~/nlidb/data/stackoverflow/sql/dev/human1_typed.txt
> mv devfile.data schedule.dev
> th buildDataPy.lua -language sql -working_dir ./s_t_3_2_rse/  -devfile ~/nlidb/data/stackoverflow/sql/test/human_akash_typed.txt
> mv devfile.data schedule.test
 
> th mrr.lua -model1 bestsql1 -model2 bestsql2 -schedule ~/nlidb/data/stackoverflow/sql/dev/mrr.json -working_dir ./s_t_3_2_rse/ 
> th mrr.lua -model1 bestsql1 -model2 bestsql2 -schedule ~/nlidb/data/stackoverflow/sql/test/mrr.json -working_dir ./s_t_3_2_rse/ 

C# MRR

> python prepare_mrr_data.py # We need to make this for dev and test

> th buildDataPy.lua -language csharp -working_dir ./c_t_2_2/   -devfile ~/nlidb/data/stackoverflow/csharp/dev/karthika_typed.txt 
> mv devfile.data schedule.dev
> th buildDataPy.lua -language csharp -working_dir ./c_t_2_2/   -devfile ~/nlidb/data/stackoverflow/csharp/test/karthika_typed.txt 
> mv devfile.data schedule.test

> th mrr.lua -model1 bestc1 -model2 bestc2 -schedule ~/nlidb/data/stackoverflow/csharp/dev/mrr.json -working_dir ./c_t_2_2/ 
> th mrr.lua -model1 bestc1 -model2 bestc2 -schedule ~/nlidb/data/stackoverflow/csharp/test/mrr.json -working_dir ./c_t_2_2/
 
** RET-IR **

All params are in this script

> cd ~/nlidb/src/core/translator
> python irMrr.py

** Comparison with Miltos **

We need to retrain on Miltos data

> th buildDataPy.lua -language miltos -working_dir ./m_t_2_2/
> th main.lua -bleufile ~/nlidb/data/miltos/so_ref.txt -gpuidx 1 -rnn_size 400 -working_dir ./m_t_2_2/  
> th buildDataPy.lua -devfile ~/nlidb/data/miltos/so_test_100.txt -working_dir ./m_t_2_2/  # I dont know why this is needed

Get the schedule from the previous section. Then,

> th mrr.lua -model1 rnn1.model1 -model2 rnn1.model2 -schedule ~/nlidb/data/miltos/mrr.json -working_dir ./m_t_2_2/  

This uses test.data

Remember that a small change needs to be made in this script because here we only have one reference. Check the script to figure it out.


** Utils **

Get typed from raw
> python ~/nlidb/src/core/sql/transform.py < stackoverflow_test_0.8.txt > test_typed.txt
> python ~/nlidb/src/core/csharp/transform.py < test.txt > test_typed.txt

Filter Csharp titles

> cd ~/nlidb/data/stackoverflow/csharp/title_filtering
> python SVM.py


