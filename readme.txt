Wormsim-2 v2.58Ap50 (Oct 15, 2018)

- fixed bug in recentlyInseminated which has been in all Wormsim versions...
- the correct implementation is: 
  (World.timenow() - inseminationdate + TimeUtils.DAY) < (factory.reproductivecycle)
  i.e. if the time since insemination plus 1 day (to prevent precision issues) is LESS THAN the reproductive cycle, 
       insemination was recent
  e.g. when insemination was 2 months ago and the reproductive cycle is 3 months: RECENT
       when insemination was 3 months ago and the reproductive cycle is 3 months: NOT RECENT

- reorganized large parts of the code to improve maintainability (especially in the Wormsim class)


Wormsim-2 v2.58Ap49 (Sep 23, 2018)
Migrated to the newer Apache Commons Math library for Continuous and Discrete Distributions
This caused the time for 1 run to increase from 0.6 to 2.0 seconds :-) !!!!!
(Specifically, the neg binomial function took 400 msecs for just 30000 times drawing 3 samples ....)
Reverted back to the much faster Colt library.
Included a stripped down version of SplittableRandom.
After a _lot_ of work back at 0.6 seconds per run.
Experimented with Java streams (for actions taken at monthly events).
In short, it appears there is no performance gain in using streams.
Parallelstreams in the handleBites function cause a crash as the event manager is not multithreaded.
Parallelstreams in e.g. summing the individual exposure do improve performance at large population size (5000) 
but use 2 cores i.e. a load of 50% on a 4 core machine and should therefore be avoided for batch runs.
Experimented with drawing random numbers one for all runs (e.g. 10^6 or 10^7).
There is no performance gain in that either as drawing a single random number (using nextDouble() and the SplittableRandom)
takes only a 3 to 4 nanoseconds (i.e. 10 CPU clock ticks) which is probably not different from getting a double from an array.
Performed 30000 runs with scenario_158.xml in 10 hours on a laptop with i7 CPU.
Expected performance on a fast CPU is less than 30 hours per scenario.

Wormsim-2 v2.58Ap48 (Sep 21, 2018)
Refactoring ageindex, agetables etc

Wormsim-2 v2.58Ap47 (Sep 20, 2018)
Reintroduced mfproduction update events for female worms (as it is faster than monthly lookup for each female worm).
ARTEFACT, NOT REAL, low FOI: Run time down to 46 seconds per run

Wormsim-2 v2.58Ap46 (Sep 20, 2018)
Fixed stupid typo in NegBinomial dist.
Run time down to 53 seconds per run.

Wormsim-2 v2.58Ap45 (Sep 16, 2018)
And now down to 0.59 seconds per run
Down to 0.69 seconds per run

Wormsim-2 v2.58Ap44 (Sep 11, 2018)

Improved to 75 secs for 100 runs due to using the G1 garbage collector -XX:+UseG1GC

79 secs for 100 runs

Added calling handleDeath() to the PopulationRegister reset() function.
This prevents zombies surviving from one run to the next.
The zombies probably did no do any harm as they are not member of the host list.
However there was one occasions (after 13000 runs) with an error that was due
to a zombies surviving to the next run....

71 secs for 100 runs

Returned to simple data structure for humans: a simple ArrayList<Human> and an ArrayList<Host>.
As Human now implements Comparable<Integer> we can quickly find a Human in a sorted ArrayList 
using binarySearch. The same is true for getting the nr of males / female in an age range;

Wormsim-2 v2.58Ap43 (Sep 10, 2018)

73 secs for 100 runs

Bugs fixed, restored handleDeath of Host (probably works also without bu tricky)
With:
ArrayList of intbirthdates
HashMap of Human (males and females)
ArrayList of hosts

Also fixed issue in reaper and mysteriously unreproducible random numbers...
This is due to SplittableRandom. Using a no arg constructor combined with setting 
the seed later is NOT the same as setting the seed in the constructor and
modifying it later.....

Wormsim-2 v2.58Ap41-fixed-dt (Sep 6, 2018)

Stack trace when dt out of range is included in .log file
Fixed two possible causes: 
a. InfectionEvent with (near) zero foi
b. BirthEvent with (near) zero total birth rate

So far, it appears that 'sometimes' (i.e. once in many many calls) the new SplittableRandom random generator
get confused by extremely large mean values for an exp dist

Wormsim-2 v2.58Ap41-debug-dt (Sep 6, 2018)

Added .errorlog with stack trace when dt goes out of range

Wormsim-2 v2.58Ap41 (Sep 5, 2018)

Added version number to info in log file.
Handled dt<0 more elegantly and included reporting in log file.
dt<0 will no longer kill a simulation run but will be fixed to a small positive value and
detailed error reporting will be in the log file.

Wormsim-2 v2.58Ap40 (Sep 4, 2018)

Run Wormsim as shown in the first line of runbatch.bat
Example parameters file: parameters.txt
Example xoiftrend file: xfoitrend_001.txt

Usage type C (used in runbatch.bat):

usage C: java --add-modules java.xml.bind -cp .;wormsim.jar;colt.jar Wormsim -runbatch -scenario=[INPUFTILE] -parameters=[PARAMETERSFILE] -xfoitrend=[XFOITRENDFILE]
         with INPUTFILE the name of the inputfile (without the .xml extent)
           and  PARAMETERSFILE the name of a text file with parameters combinations, one per line and each line containing:
                -a=[false|true] to indicate appending the output
                -seed=[seeds] with [seeds] a comma separated list of seeds e.g. -seed=0,1,2
                -rbr=[rbrs] with [rbrs] a list of relative biting rate values e.g. -rbr=0.7,1.0,1.2
                -pop=[popmult] with [popmult] a list of population size multipliers e.g. -pop=1,2,5
                -exp=[exp] with [exp] a list of list of exposure heterogeneity parameters e.g. -exp=0.7,0.8,0.9
                -f=[frac] with [frac] a list of fractions of the population to be sampled at each survey e.g. -f=0.333
                -n=[nsamples] with [nsamples] the number of population samples e.g. -n=3
                -xfoimult=[xfoimult] with [xfoimult] alist of multiplier for the trend in external FOI e.g. xfoimult=0.8,1,1.2
             a full example of the first line in the parameters file:
                -a=false -seed=0,1,2,3,4,5,6,7,8,9 -rbr=0.5,0.6 -pop=1,2 -exp=0.1,0.2 -xfoimult=1,2 -f=0.333 -n=3
                this means that 10 * 2 * 2 * 2 * 2 simulation runs will results from the above single line
             another simpler example:
                -a=true -seed=0 -rbr=0.6 -pop=1 -exp=0.2 -xfoimult=1
                meaning a single simulation run, no samples (just the entire population)
             or even"
                -a=true -rbr=0.6 -pop=1 -exp=0.2 -xfoimult=1
                a single simulation run with random seed
           and  XFOITRENDFILE the name of a text file with a trend in external FOI in one line
           e.g. 1851.99:1,1999.99:1,2004.99:2,2009.99:3
           NOTE that only the first line of XFOITRENDFILE will be read as a trend is specific for a scenario


Further performance improvements: now 0.70 secs per run on an i7-8550U due to using a modified version of SplittableRandom.

Wormsim-2 v2.58Ap39 (Sep 1, 2018)

Further performance improvements: now 0.79 secs per run on an i7-8550U

Wormsim-2 v2.58Ap38 (Aug 30, 2018)

Fixed a concurrentModificationException caused by walking a list of worms for treatment and killing some at the same time...

Wormsim-2 v2.58Ap37 (Aug 27, 2018)

Performance improvements: from 212 secs for 130 runs to 118 secs i.e. about 0.85 to 0.9 seconds per run on a i7-8550U 

Modifications:
a. The reaper: if pop size > max pop size, a number of people equal to the specified fraction will be reaped
   This works slightly different from the previous method, which was (if pop size > max pop size) to draw a random number 
   between 0 and 1 for each person and if less than the specified fraction, reap that person (i.e. equivalent to drawing 
   the number to be reaped from a binomial distribution (by the way for each gender separately)
b. for a female worm, 'recentlyInseminated' was determined by:
        //double dt = World.timenow() - inseminationdate;
        //int nmonths = (int) Math.round(dt/util.Util.MONTH);
        //return nmonths <= (int) Math.round(factory.getReproductiveCycle()/util.Util.MONTH);
    it is simpler now:
        (World.timenow() - inseminationdate + Util.DAY) <= factory.reproductivecycle;
    NOTE: need to check if there is a consistent difference between the two methods (due to rounding etc)
c. Many simplifications in the code: removed many (abstract) classes, made 'sex' an attribute of Human 
   (instead of Male and Female descendent classes), removed old Schisto code. 
   These simplifications remove clutter and make the code more readible.
   Also, optimized the code executed at a monthly event, i.e. prevented as much as possible multiple passes over the collection of hosts.
   These optmimizations may make the code less readible.
d. Use Java 10

Wormsim-2 v2.58Ap36 (Aug 23, 2018)

To run a batch of parameter combinations run Wormsim with for instance:
java -cp .;wormsim.jar;colt.jar Onchosim2.Wormsim -runbatch -parameters=parameters.txt -scenario=example_001 -xfoitrend=xfoitrend_001.txt

The first lines of parameters.txt:
-a=false -seed=0,1 -rbr=0.5,0.6 -pop=1 -exp=0.1,0.2 -f=0.3 -n=0 -xfoimult=1.0

Optional are:
-seed
-f 
-n



Example contents of xfoitrend.txt
1999.99:1,2004.99:2,2009.99:3

Output will be written to 
example_001.mflog
example_001.uflog (a summarized version of mflog)
example_001.log (the logfile)

Both the mflog and uflog files will be zipped and the original file will be deleted


wormsim-2 v2.58Ap35 (Aug 17, 2018)

To run a batch of parameter combinations run Wormsim with for instance:
-runbatch -parameters=parameters.txt -scenario=Master_template_Lf_ap27

i.e. with the runbatch option, followed by the name of a text file that contains parameters or parameter combinations per line.
The first lines of parameters.txt:
-a=false -seed=0,1 -rbr=0.5,0.6 -pop=1 -exp=0.1,0.2 -f=0.3 -n=0 -xfoi=1999.99:1,2004.99:2,2009.99:3
-a=true  -seed=2,3 -rbr=0.5,0.6 -pop=1 -exp=0.1,0.2 -f=0.3 -n=0 -xfoi=1999.99:1,2004.99:2,2009.99:3

Output will be written to 
Master_template_Lf_ap27.mflog

wormsim-2 v2.58Ap34 (Aug 16, 2018)

added option to run from a text file with command lines
see batch-run-from-file.bat and cmdlines.txt


wormsim-2 v2.58Ap33 (Aug 15, 2018)

- extended log output with 6/7 year olds

wormsim-2 v2.58Ap32 (Jul 12, 2018)

1. Modified new command line option '-runbatch' with the following parameters:
   -a=[true|false] -i=[inputfile] -s=[seeds] -rbr=[rbrs] -pop=[popsizes] -exp=[expp1] -f=[samplefraction] -n=[nsamples] -xfoi=[xfois] -xfoiA=[xfoiA] -decayA=[paramsA] -xfoiB=[xfoiB] -decayB[=paramsB] 
   with:
        [true|false] append output to mflog e.g. -a=false

        [inputfile] the name of the inputfile (without extent) e.g -i=example
        [seeds]    a comma separated list of seed numbers e.g. -s=1,2,3");
        [rbrs]     a comma separated list of relative biting rates e.g. -rbr=0.5,0.6,0.7");
        [popmult]  a comma separated list of population size multipliers e.g. -pop=1,2,5");
        [p1]       a comma separated list of exposure heterogeneity parameters p1 e.g. -exp=0.7,0.8,0.9");
        [frac]     the fraction of the population to be sampled at a survey e.g. -f=0.2");
        [samples]  the number of population samples at a survey -n=10");
        [xfois]    a comma separated list of pairs of time:xfoi e.g. -xfoi=2000:1,2001:1.4;2010:2.0
                   -xfoi overrides the options -xfoiA and -xfoiB and replaces values specified in the inputfile from the 1st year specified
        [xfoiA]    a comma separated list of external FOI values e.g. -xfoiA=2,4,6");
        [paramsA]  a comma separated list of decay parameters for the xFOI_A values e.g. -decayA=2010,0.8,1,0.1");
                  with subsequent values for ta,fa,labda1a,labda2a leading to");
                  xfoiA(t-ta) = xfoiA * (fa*exp(-labda1a*(t-ta)+(1-fa)*exp(-labda2a*(t-ta)) for tb>t>=ta");
        [xfoiB] and [paramsB] analoguously to the A params and leading to");
                  xfoiB(t-tb) = xfoiB * (fb*exp(-labda1b*(t-tb)+(1-fb)*exp(-labda2b*(t-tb)) for t>=tb");
        -o: mimickold

An example of a valid command line:
java -cp .;wormsim.jar;colt.jar Onchosim2.Wormsim -runbatch -a=false -i=Master_template_Lf_ap27 -seed=0,1 -rbr=0.5 -pop=1 -exp=0.1,0.2 -f=0.3 -n=3 -xfoiA=2,3 -decayA=2010,0.8,1,0.1 -xfoiB=2,3 -decayB=2015,0.8,1,0.1

It is of course possible to put multiple command lines in a batch file.
See batch-run.bat (Windows) and batch-run.sh (Unix).
NOTE: Make sure to save batch-run.sh from the nano editor on Linux in MS-DOS mode (i.e. with CR LF line endings). 
      Somehow Java running on Linux does not seem to honor the platform specific line endings (i.e. just a LF on Unix, and CR LF on DOS)


wormsim-2 v2.58Ap30 (Jul 4, 2018)
1. Allowed values or rbr > 1
2. Set maximum mbr to 1.000.000 (modified from 10.000)
3. Added a new command line option '-runbatch' with the following parameters:
   -s[seeds] -b[rbrs] -p[popsizes] -k[expp1] -l[labdas] -t[year] -x[fraction] -n[nsamples]
   with:
   [seeds]   : a comma separated list of seeds e.g. -s1,3,5,7
   [rbrs]    :  a comma separated list of relative biting rates e.g. -b0.6,0.7,0.8
   [popsizes]: a comma separated list of population size multipiers e.g. -p1,2,5
   [expp1]   : a comma separated list of p1 values of the exposure heterogeneity distribution (applied to both M and F) e.g. -k1,2,3
   [labdas]  : a comma separated list of labda values i.e. rates [1/year] of decay of the external force of infection e.g. -l1,2
   [year]    : start moment of decay in external force of infection e.g. -t2010
   [fraction]: the fraction of the population to be sampled at survey moments e.g. -x0.2
   [nsamples]: the number of samples at surveys moments e.g. -n5
An example of a valid command line:
java -cp .;wormsim.jar;colt.jar Onchosim2.Wormsim -runbatch -iMaster_template_Lf_ap27 -s1,3,5,7 -b0.5,0.6,0.7 -p1,2,4 -k0.1,0.2,0.3 -x0.3 -n3 -l1,2 -t2010

wormsim-2 v2.58Ap29 (Jun 12, 2018)
1. compiled with JDK 1.7 (again, as AWS AMI had Java 7)
2. TO DO: remove restriction in biting rate 
3. TO DO: profile performance

wormsim-2 v2.58Ap28 (Jan 17, 2018)
1. This version is functionally identical to version 2.58Ap27.
2. The only difference is that this version has been compiled with Java 7 and should therefore run on JRE 7.

wormsim-2 v2.58Ap27 (Oct 15, 2017)
1. Fixed bug in AggregatedDetailedOutputWriter that caused incomplete Z file output ; this was due to not flushing an OutPrintWriter
2. Added the -u option to not produce zipfiles

wormsim-2 v2.58Ap26 (Oct 5, 2017)
1. Added column for external FOI in standard output
2. Fixed bug that caused any other runs than the 1st run to attain zero prevalence after run-in period
   ( which was due to not resetting vector control to 1.0 at the start of each run)
3. Added mf5+ values 

wormsim-2 v2.58Ap25 (Feb 23, 2017)
Fixed bug in output on anywpos.
Corrections of previous results can be made by:
subtracting the M_wpairpos number from M_anywpos
subtracting the F_wpairpos number from F_anywpos

wormsim-2 v2.58Ap24 (Dec 7, 2016)

Added OV16 output (i.e. serology)
See the *Z.### and *Z.txt files.
Explanation of the header of the *Z.txt files:
year: time of survey
age: upper limit of age group
OV16[SEX][FLAGNR][-/+]: number of people of that SEX with FLAGNR positive (+) or negative (-)

OV16 FLAGNRs:

1: positive if number of prepatent worms > 0
2: positive if number of    patent worms > 0
3: positive if mf density > 0


Use the -d command-line option to enable detailed output.

wormsim-2 v2.58Ap23 (Sep 22, 2016)

The initial FOI:
<initial.foi duration="5" foi="10.0"/>
has been replaced by:
            <external.foi>
                <start year="1790" month="0" foi="4" exclusive="true"/>
                <start year="1797" month="6" foi="0" exclusive="false"/>
            </external.foi> 

wormsim-2 v2.58Ap22 (Sep 13, 2016)

Fixed bug that caused exposure / contribution interventions not to be reset at the start of the next simulation run 
(i.e. in practice from the 2nd run (runnr 1)).

wormsim-2 v2.58Ap21 (Sep 7, 2016)

Corrected code to comply with appendix to STH paper: Modeling of soil-transmitted helminths in Wormsim
Specifically
- redefined L1uptake (now just the total L1uptake of the population, i.e. no longer the mean)
- divided the FOI applied to each individual by dividing by the summed population exposure 

wormsim-2 v2.58Ap20 (Sep 5, 2016)

Moved "fraction.excluded" from the XML compliance element to the exposure and contribution interventions, 
and edited the code where necessary.

<exposure.interventions>
  <!-- note that effectivity below means the impact on individuals that are not excluded -->
  <moment year="1999" month="0" effectivity="0.75" fraction.excluded="0"/>
  <moment year="2002" month="6" effectivity="0.75" fraction.excluded="0"/>
</exposure.interventions>
<contribution.interventions>
  <!-- note that effectivity below means the impact on individuals that are not excluded -->
  <moment year="1999" month="0" effectivity="0.75" fraction.excluded="0"/>
  <moment year="2002" month="6" effectivity="0.75" fraction.excluded="0"/>
</contribution.interventions>

Also, to ensure consistency, also moved "fraction.excluded" from the XML compliance element for mass treatment rounds:
        <mass.treatment>
            <treatment.rounds>
                <treatment.round year="2000" month="0" coverage="0.5563910" delay="-1" fraction.excluded="0"/>
                <treatment.round year="2000" month="1" coverage="0.6390977" delay="-1" fraction.excluded="0"/>
                <treatment.round year="2000" month="7" coverage="0.5939850" delay="-1" fraction.excluded="0"/>
                <treatment.round year="2000" month="8" coverage="0.5413534" delay="-1" fraction.excluded="0"/>
            </treatment.rounds>
            <compliance fraction.malabsorption="0" compliance.model="0">
                <age.and.sex.specific.compliance age.limit="2" male.compliance="0" female.compliance="0"/>

Note that the default value of fraction.excluded equals zero.
Therefore, it is allowed to not specify fraction.excluded in the individual exposure and contribution interventions, 
and in the mass treatments rounds.
(As an aside, "fraction.excluded" is also defined for vector control moments but not used in the code.)


wormsim-2 v2.58Ap19 (Jul 27, 2016)

Changed the implementation of vector control (and exposure / contribution control).
Instead of defining a period of vector control, vector control is simply updated at specified moments:
    <vector.control>
        <moment year="1981" month="0" effectivity="0.41"/>
        <moment year="1981" month="3" effectivity="0.82"/>
        <moment year="1981" month="6" effectivity="0.47"/>
        <moment year="1982" month="0" effectivity="0.25"/>
        <moment year="1982" month="3" effectivity="0.00"/>
    </vector.control>
It is therefore necessary to reset vector control (e.g. to 0) at the end of a (set of) period(s) of vector control.
Note that the output on mbr shows the effect of vector control during the past month, 
as surveys are executed just before the end of the month (depending on delay which is usually set to -2 (hours)).



wormsim-2 v2.58Ap18 (Mar 2, 2016)
  Added support for multiple types of treatments.
  
  The inputfile now expects (see v258Ap18_lymfasim_india.xml for an example):
  <mass.treatments>
    <treatment.effect.variability dist.nr="0" mean="1.0"/>
    <mass.treatment>
      <treatment.rounds>
        .
        .
      </treatment.rounds>
      <compliance fraction.excluded="0.0" fraction.malabsorption="0.0">
        .
        .
      </compliance>
      <treatment.effects permanent.reduction.mf-production="0.01" period.of.recovery="0.01" shape.parameter.recovery.function="1.0" fraction.killed="0.5">
        <fraction.mf.surviving dist.nr="0" mean="0.01"/>
      </treatment.effects>
    </mass.treatment>
    <mass.treatment>
     .
     .
    </mass.treatment>
  </mass.treatments>
    
  In other words: 
  <treatment.effect.variability> is common to all mass treatments
  Multiple sets of <mass.treatment> are supported, each with its own treatment rounds, compliance, and treatment effects

  It is even possible to mix treatment types (i.e. in years 2000, 2002, 2005 treatment A and in 2001, 2003, 2004 treatment B).
  The number of different treatment types is unlimited (except for the imagination of the user).


wormsim-2 v2.58Ap17 (Feb 6, 2016)
  fixed bug that caused NaN in output (due to dividing 0/0 in AbstractHostCollection.getMeanL1Uptake())
wormsim-2 v2.58Ap16 (Feb 4, 2016)
  fixed bug that caused exposure and contribution interventions not to work
wormsim-2 v2.58Ap15 (Jan 31, 2016)
  additional output (by sex and age class) on any worm+, female worm+, and worm pair+
  for now, only in the X files, and for PATENT worms
wormsim-2 v2.58Ap14 (Jan 28, 2016)
  bednets and bug fix
  intermediate version - because of issues with coverage model 0 ....

wormsim-2 v2.58Ap13 (Jan 19, 2016)
  bednets etc
  intermediate version - because of issues with coverage model 0 ....

wormsim-2 v2.58Ap12 (Jan 12, 2016)
Fixed bug (forgot to regenerate XML classes with xjc resulting in 1.0 default for fraction mf surviving)
wormsim-2 v2.58Ap11 (Jan 12, 2016)
In the first xml element use "lymfasim" to enable exponential decay of mf survival:
<wormsim.inputfile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                   xsi:noNamespaceSchemaLocation="wormsim.xsd"
                   model="lymfasim">
and use mf-survival instead of mf-lifespan:
    <worm mf-survival="0.9" monthly.event.delay="+1">
to specify the fraction surviving to the next month

wormsim-2 v2.58Ap10 (Jan 6, 2016)

Implemented both anti-L3 and anti-worm (i.e. fecundity) immunity
Specify half-time (in years) of anti-L3 or anti-worm immunity with TH.L3 and TH.W
See further the formal description of Lymfasim.
Gamma is the strength of the immunity (either anti L3 or anti-worm)
and immunity.index is the individual variation in immune response

    <anti.L3.immunity TH.L3="1" gamma="0.1">
        <male>
            <immunity.index dist.nr="4" min="0" max="20" mean="1.0" p1="2"/>
        </male>
        <female>
            <immunity.index dist.nr="4" min="0" max="20" mean="1.0" p1="2"/>
        </female>
    </anti.L3.immunity>
    <anti.Worm.immunity TH.W="1" gamma="0.1">
        <male>
            <immunity.index dist.nr="4" min="0" max="20" mean="1.0" p1="2"/>
        </male>
        <female>
            <immunity.index dist.nr="4" min="0" max="20" mean="1.0" p1="2"/>
        </female>
    </anti.Worm.immunity>

A new additional parameter is success.ratio in 
    <fly transmission.probability="0.07345" success.ratio="0.00302">
This was necessary to boost anti-L3 immunity using mtp rather than the individual foi (without immune suppression)

wormsim-2 v2.58Ap9 (Mar 20, 2015)

Specify in the inputfile either:
    <skin.mf-density.per.worm fun.nr="1" a="7.6" c="-1"/>
or (see v258Ap9_STH_sat_fn.xml):
    <alt.skin.mf-density.per.worm>
            <a dist.nr="4" mean="1" p1="0.5"/>
            <b dist.nr="4" mean="1" p1="0.5"/>
            <c dist.nr="0" mean="1"/>
    </alt.skin.mf-density.per.worm>
In the latter case, the following formula is used to calculate sl(t) as a 
function of el(t), a, b and c:
 sl(t) = c * b * a * el(t) / (a*el(t)+b)

Note that the new saturating function for sl(t) is equivalent to the well-known equation 
Michaelis-Menten equation for enzyme kinetics: V = Vmax * [S] / (Km + [S])
(with Vmax = b*c and Km=b/a)

wormsim-2 v2.58Ap8 (Mar 19, 2015)
* modified as follows
  lu(m)_res=lu(m)_in+psi*(lu)(m-1)_res
  lr(m)    =lu(m)_res*zeta*v
* lines in mf log now contain: 
  seed year mf+ mf5+ w+ N aNmf aNmf20 N20 CMFL

wormsim-2 v2.58Ap7b (Mar 5, 2015)
* fixed bug in zeta and psi effects
  implemented:
  lu(m)_res=lu(m)_in+(1-zeta)*psi*(lu)(m-1)_res
  lr(m)    =lu(m)_res*zeta*v

wormsim-2 v2.58Ap7 (Mar 4, 2015)
* STH: added zeta and psi
* option to define skin snip categories
* see v258Ap7_test_1.xml

wormsim-2 v2.58Ap6 (Feb 25, 2015)
* lines in mf log now contain: seed year mf+ w+ N aNmf20 N20 CMFL

wormsim-2 v2.58Ap5 (Feb 18, 2015)
* STH extensions:

  renamed the <exposure> element to <exposure.and.contribution> and included a
  contribution function for age dependent contribution and an optional element 
  contribution index.

    <exposure.and.contribution>
        <initial.foi duration="7.5" foi="4.0"/>
        <male>
            <exposure.function fun.nr="1" a="0.05" c="1"/>
            <exposure.index dist.nr="4" min="0" max="20" p1="3.865"/>
            <contribution.function fun.nr="1" a="0.05" c="1"/>
        </male>
        <female>
            <exposure.function fun.nr="1" a="0.035" c="0.7"/>
            <exposure.index dist.nr="4" min="0" max="20" p1="3.865"/>
            <contribution.function fun.nr="1" a="0.035" c="1"/>
        </female>
    </exposure.and.contribution>

  add labda to age.dependent.mf-production to allow for density dependence of mf production
  the default value is 0 (also when labda attribute is omitted):

        <age.dependent.mf-production labda="0">
   

wormsim-2 v2.58Ap4 (Feb 12, 2015)
* added 'extra surveys' option
* added commandline option -n to suppress all output except the error log and mf log
* added mf log
* replaced definition of 'warm-up' by simulation start year

wormsim-2 v2.58Ap3 (Feb 13, 2014)
* fixed bug that caused FOI to become slightly negative resulting in dt<0
wormsim-2 v2.58Ap2 (Feb 13, 2014)
* fixed bug that caused the outputfile of the last succesful run to be replicated while averaging
* also fixed issue with the extent of outputfiles that occurred when run numbers do not start at 0
 
wormsim-2 v2.58Ap1 (September 25, 2013)
* fixed bug in OCP standardized output
wormsim-2 v2.58A (March 20, 2013)
* the detailed output of individual runs (*X.nnn and *Y.nnn files in the zip) are now 
  plain tab delimited text files
wormsim-2 v2.58 (Jan 7, 2012)
* added a new section to the inputfile to allow defining age classes for survey output 
  different from those used to specify the standard population
  see rbr75-exp677c13.xml and rbr75-exp677c14.xml and of course the XML schema wormsim.xsd
wormsim-2 v2.57 (Jan 5, 2012)
* modified implementation of ivermectin treatment effect to be compliant with Onchosim
  an ivermectin treatment can never lead to a recovery of a worm sooner than 
  the recovery from a previous treatment (this could happen due to heterogeneity in treatment effect and
  small interval between ivermectin treatments)
* corrected an error in the averaging procedure that caused error in zipping and deleting 
  output of individual runs when a range of runs did not start with 0

wormsim-2 v2.56 (Jan 4, 2012)
* corrected an error in the sequence of events triggered by the monthly event
  until now, L1uptake was based on mf load of the previous month
  in Onchosim, as now in Wormsim, the order is as follows:
  1. reproduction (i.e. insemination of female worms)
  2. mf production update
  3. calculcate FOI from L1uptake (or use clamped FOI during warmup period)
  4. distribute new worms

wormsim-2 v2.55 (Jan 3, 2012)
* corrected an error in the implementation of the delay of the monthly event
* inputfile rbr75-exp677c6.xml has the correct delays:
  reaper   		-4
  newborns 		-3   
  survey  		-2
  ivermectine 	-1
  monthly event +1
  
wormsim-2 v2.54 (Jan 2, 2012)
* included prepatent worms (both M and F) in ivermectin treatment ; this
  will cause F worms to have a lower mf production when becoming patent and inseminated 
  the first time


wormsim-2 v2.53 (Jan 2, 2012)
* modified defaults for optional delays; omitting the delays mentioned below will result in the default values specified below.

* modified default for optional delay attribute to survey start 		(default = -5 hours)
see:	<surveillance nr.skin-snips="2">
            <start year="2000" delay="-5"/>
            <stop year="2020"/>
            <interval years="5"/>
        </surveillance>

* modified default for optional delay attribute to treatment rounds		(default = -4 hours)
see:	<mass.treatment>
              <treatment.rounds>
                     <treatment.round year="2000" month="2" coverage="0.6" delay="-4"/>
                     <treatment.round year="2001" month="2" coverage="0.6" delay="-4"/>
                     <treatment.round year="2002" month="2" coverage="0.6" delay="-4"/>

* modified default for optional delay attribute to fertility table 		(default = -3 hours)
see:	<fertility.table delay="-3">

* modified default for optional delay attribute to the reaper   		(default = -2 hours)
see: 	<the.reaper max.population.size="440" reap="0.1" delay="-2"/>

* added optional monthly.event.delay attribute to worm					(default = -1 hour)
  this affects the monthly worm distribution
see		<worm mf-lifespan="9" monthly.event.delay="-1">

The allowed range for these attributes is +/- 12 (hours)

wormsim-2 v2.52 (Nov  30, 2011)
* included the Onchosim erroneous calculation Cw' = Cw + fc (instead of - as specified in the manual Cw' = Cw/(1-fc)) in the -o option
* added optional delay attribute to survey start 	(default = +0 hour)
* added optional delay attribute to the reaper   	(default = +1 hour)
* added optional delay attribute to fertility table (default = +2 hours)
* added optional delay attribute to treatment rounds(default = +3 hours)
  the allowed range for these attributes is +/- 12 (hours)
* allowed monthnr = 0 which is also the default for surveys ; yearnr=2000 and monthnr=0 is the same as yearnr=1999 monthnr=12


wormsim-2 v2.51 (Nov  9, 2011)
* added command-line option (-o) to reproduce Onchosim errors in births and exposure of newborns:
  with the -o option, newborns will be generated at the end of each year and added to the population in the past year (after the yearly survey)
  
wormsim-2 v2.50 (Oct 12, 2011)
* corrected error in getProduction() for Onchosim. The error was that recentlyInseminated() was NOT checked. 


wormsim-2 v0.01 (May 3, 2011)
* created a common code base for onchosim and schistosim
* checked events package: current version of event package will be common base
* see diff-oncho-schisto.txt file in package directory


Wormsim v0.96
1. Cosmetic change in Host.java: added method getSexRatio().
2. Modified FemaleWorm.recentlyInseminated() which did not produce the same result when 
   handling ReproductionEvent and MfProductionUpdateEvent. This may explain observed differences
   between Onchosim-97 and Wormsim-0.94 reported by Luc Coffeng.  

Wormsim v0.95
1. De random number generator wordt nu ook gebruikt voor poisson en neg binomiale verdelingen. 
   Dat betekent dat een run met dezelfde seed en inputfile altijd hetzelfde resultaat oplevert.
2. De malabsorptie factor is toegevoegd voor ivermectine behandelingen (random, niet consistent).
3. De leeftijdsafhankelijke mf productie moet nu in hetzelfde format als bij Onchosim worden opgegeven 
  (ipv de leeftijd vd worm moet nu het aantal jaar sinds patent worden gekoppeld aan een mf productie factor)
4. Een simulatierun duurt nu 4 sec ipv 20 sec (bij een maximale populatiegrootte van 440 op een enkele jaren oude laptop). 
 
