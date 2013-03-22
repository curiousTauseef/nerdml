nerdml
=========

This project contains a list of scripts used to format the ouput of NER parsers such as 
NERD, Stanford-CRF and Ritter's UW_Twitter_NLP, as input of a machine learner algorithm.

3rd party libraries used in these scripts:
- Weka: http://www.cs.waikato.ac.nz/ml/weka/
- Stanford-CRF: http://nlp.stanford.edu/software/CRF-NER.shtml
- Ritter's UW_Twitter_NLP: https://github.com/aritter/twitter_nlp


#### Documentation
This documentation explains how to create machine learning datasets out of the NERD, Stanford and UW_Twitter_NLP tools output. 
The commands in this file assume that there are 10 folders named 1 to 10 that each contain one part of the dataset for 10-fold cross validation.

##### input
a CSV file (nerdANDstanfordANDuwtwitternlp.mconll), where the columns are: 
1st:       token
2nd:       GS
3rd..12th: NERD_i parser
13th:      Stanford-CRF
14th:      Ritter's UW_Twitter_NLP

##### preprocessing
Create input file for pos tagger (make sure the file has two columns, even if the second is only a dummy column, otherwise the tagger will choke)   

    cd cross_validation ;
    # the pos tagger assumes proper hashtags and urls, so insert some dummy values here
    for x in {1..10} ; do cd $x ; gcut -f1,2 -d" " < nerdANDstanfordANDuwtwitternlp.mconll | gsed 's/_Mention_/\@blabla/g ; s/_URL_/http:\/\/www.blabla.com/g ; s/_HASHTAG_/\#pgroth/g' | gtr " " "\t" > nerdANDstanfordANDuwtwitternlp_inputForPOS ; cd .. ; done
    # also put second part of the file somewhere 
    for x in {1..10} ; do cd $x ; gcut -f2- -d" " < nerdANDstanfordANDuwtwitternlp.mconll > nerdANDstanfordANDuwtwitternlp_complementToPOS ; cd .. ; done
    # run the pos tagger (check the location of the tagger!)
    for x in {1..10} ; do cd $x ; ./../../../ark-tweet-nlp-0.3.2/runTagger.sh --input-format conll nerdANDstanfordANDuwtwitternlp_inputForPOS | gcut -f1,2 | gtr "\t" " " | gsed 's/@blabla/_Mention_/g ; s/http:\/\/www.blabla.com/_URL_/g ; s/\#pgroth/_HASHTAG_/g' > nerdANDstanfordANDuwtwitternlp_postagged.conll ; cd .. ; done
    # glue the files together 
    for x in {1..10} ; do cd $x ; paste -d" " nerdANDstanfordANDuwtwitternlp_postagged.conll nerdANDstanfordANDuwtwitternlp_complementToPOS > nerdANDstanfordANDuwtwitternlp_POStaggedInputForPostProcessingRules.mcoll ; cd .. ; done 


Add naive gazetters. Run rules (_URL_ can't be an entity etc )Check the location of the RunNERDPostprocessingRules.pl script and adjust the path if necessary  

    for x in {1..10} ; do cd $x ; perl ../../../RunNERDPostprocessingRules.pl nerdANDstanfordANDuwtwitternlp_POStaggedInputForPostProcessingRules.mcoll > nerdANDstanfordANDuwtwitternlp_POStaggedPostProcessedInputForMLFeatureGeneration.mcoll ; cd .. ; done
    

Align with GS to reinsert _ENDOFTWEET_ tokens (needed for some ML features)

    for x in {1..10} ; do cd $x ; perl ../../../alignGoldStandardWithNERDOutput.pl validation.GS nerdANDstanfordANDuwtwitternlp_POStaggedPostProcessedInputForMLFeatureGeneration.mcoll | cut -f3 | sed 's/\%/percent/g'  > nerdANDstanfordANDuwtwitternlp_POStaggedPostProcessedInputForMLFeatureGeneration_aligned.mcoll ; cd .. ; done  


Add ML features 

    for x in {1..10} ; do cd $x ; perl ../../../AddMLFeaturesAndCLeanUp.pl nerdANDstanfordANDuwtwitternlp_POStaggedPostProcessedInputForMLFeatureGeneration_aligned.mcoll > ../../MachineLearningExperiments/nerdANDstanfordANDuwtwitternlpANDmlFeatures_Part$x.mcoll ; cd .. ; done 
    
    
#### Licence
These scripts are free software; you can redistribute it and/or modify it
under the terms of the GNU General Public License published by
the Free Software Foundation, either version 3 of the License, or (at 
your option) any later version. See the file Documentation/GPL3 in the
original distribution for details. There is ABSOLUTELY NO warranty. 
