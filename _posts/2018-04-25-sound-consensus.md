---
layout: post
title:  "Sound Consensus"
date:   2018-04-25
categories: datamining music streaming
math: true
thumbnail: /images/musicMining/8.png
---


Introduction
=============

As part of my Data Mining class at the University of Utah I teamed up with [Alex BlackBurn][alex-blackburn] and [Hoachen Zhang][hoachen-zhang] to analyze music. Our goal was to expand music theory through the application of various data mining techniques. We split our efforts, and I focused on finding the most common melodic patterns.

Feel free to skip to the [results](#Results).

Data 
=====

We used a [music corpus][midi-files] of midi files found on reddit. Uncompressed the 130,000 songs take up about 3.5 Gb of space. Midi is a compressed form of binary data used to describe music and events for a wide variety of electronic musical instruments. It consists of a stream of events with the majority of messages being "Note on" or "Note off" events. For a different project Alex had written a Java program that transforms the streams into lists of notes. I then transformed the list of notes to a list of intervals. For example B, C, D, E becomes 1, 2, 2. I used intervals because it is key agnostic. So, the same sounding melodic pattern played in different registers were transformed to the same pattern.

Finally, we split each list into k-grams. K-grams are a common construct in NLP, which can be imagined as a sliding a window of size k over sequence. For example "music is great" with k = 2 yields the 2-grams "music is" and "is great" and "1, 2, 2" gives the 2-grams "1,2" and "2,2". This is process is artificial, as  melodic phrases do not neatly have k number of notes, but it is efficient, and the process of picking out melodic phrases was outside the scope of this project. 

Process
=========

As most of the events in the midi files are either 'notes on' or 'note off', each taking 16 bytes,and with a data set size of 3.5 Gb there are approximately 13,671,000 notes. The Java Note class needs about 16 bytes for overhead and has 12 ints, 7 Strings, 2 longs, and 1 float fields. So $$ 16 + (12 * 16) + (7 * 64) + (2 * 32) + 32 =  752$$ bytes are required per note. Which gives a total size of 
$$ (752 *13671000) / 1000000000 = 10.28$$ Gb.

As this was larger then my computers available memory, and it was a Data Mining course so it was better to do something exciting, I decided to go with a streaming approach. I focused on a few standard frequent item approaches, MisraGries, SpaceSaving, and Lossy Counting. 
Using Maycon Viana Bordin's Java streaming library [streaminer][streaming-algo] the simplified code below shows the process from start to finished.

{% highlight java %}
public static void main(String[] args){

    File corpus = new File("corpusPath");

    CorpusAnalyzer analyzer = new CorpusAnalyzer(); //Used to read in midi files
    MidiReader reader = new MidiReader(analyzer);  //Used to read in midi files

    int tupleSize = 8;
    int counters = 1000;

    // This does the actual Counting
    MisraGries<List<Integer>> frequencyCounts = new MisraGries<List<Integer>>(counters);

     for (File file : corpus.listFiles()) {

        //transforms midi to Notes
        ArrayList<ArrayList<Note>> tracks = reader.readSequence(file, -1, false, 16);
        
        //transform note to intervals
        for (ArrayList<Note> track : tracks) {
            ArrayList<Integer> intervalList = new ArrayList<>();
            for (int i = 1; i < track.size(); i++) {
                int interval = track.get(i).key - track.get(i - 1).key;
                intervalList.add(interval);
            }

            //Counts frequent items
            for (int j = 0; j < track.size() - tupleSize; j++) {
                List<Integer> kGram = intervalList.subList(j,j+tupleSize);
                frequencyCounts.get(i).add(kgram);
            }
        }
    }

    frequencyCounts.getFrequentItems(minSupport);
}

{% endhighlight %}

MisraGries worked the best as far as time and results, while SpaceSaving never finished. LossyCounting worked, but the results where nearly the same as MisraGries and it took twice as long.

Results {#Results}
========

For all trials, and all k-gram lengths, using all algorithms, the most commonly occurring pattern was the same note repeated over and over again. The rest of the significant results consisted of two notes repeated back and forth. Though not very exciting it makes sense as those patterns make up most harmony lines, and the system couldn't tell the difference between harmony and melody. I used a few heuristics to try and make things better, such as counting each k-gram only once per song or removing tracks which were highly repetitive, but each time the results were nearly same. All final result data can be found [here][final-data].

There were some interesting patterns but they had relatively low counts. I've plotted a few on a staff below.

![p5]
![p6]
![p7]
![p8]

For those interested our final poster for the project can be found [here][poster].


[final-data]: /data/musicMiningFinalResults.tar.gz
[alex-blackburn]: https://www.linkedin.com/in/alex-blackburn-b96435116/
[hoachen-zhang]: https://www.linkedin.com/in/haochen-zhang-6b07b7117/
[midi-files]: https://www.reddit.com/r/WeAreTheMusicMakers/comments/3ajwe4/the_largest_midi_collection_on_the_internet/
[streaming-algo]: https://github.com/mayconbordin/streaminer
[poster]: https://docs.google.com/presentation/d/16bm_XPib0WB3Gg32vKQfGnAA_wBymIKr_dvl2h3as-c/edit?usp=sharing
[p5]: /images/musicMining/5.png
[p6]: /images/musicMining/6.png
[p7]: /images/musicMining/7.png
[p8]: /images/musicMining/8.png
