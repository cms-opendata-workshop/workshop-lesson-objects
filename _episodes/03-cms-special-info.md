---
title: "Accessing CMS-specific object information"
teaching: 15
exercises: 30
questions:
- "Where can I find CMS-specific information about a given object, like identification criteria?"
objectives:
- "Bookmark informational web pages for different objects"
- "Learn member functions for identification and isolation of objects"
- "Learn member functions for detector-related quantities"
- "Practice accessing and saving these quantities"
keypoints:
- "Physics objects in CMS are reconstructed from detector signals and are never 100% certain!"
- "Identification and isolation algorithms are important for reducing fake objects."
- "Member functions for these algorithms are documented on public TWiki pages."
---

When is a muon not a muon? How do we guarantee that signals from the muon detector correspond to
a "real" muon? How do we distinguish between muons produced directly from decays of more massive
particles (ex: $H \rightarrow 4\mu$) from muons produced in semi-leptonic decays of heavy quarks?

The most signicant difference between a list of certain particles from a Monte Carlo generator and a list
of the corresponding physics objects from CMS is likely the inherent uncertainty in the reconstruction.
Selection of "a muon" or "an electron" for analysis requires algorithms designed to separate "real"
objects from "fakes". These are called **identification** algorithms.

Other algorithms are designed to measure the amount of energy deposited near the object, to determine
if it was likely produced near the primary interaction (typically little nearby energy), or from the
decay of a longer-lived particle (typically a lot of nearby energy). These are called **isolation**
algorithms. Many types of isolation algorithms exist to deal with unique physics cases!

Both types of algorithms function using **working points** that are described on a spectrum from
"loose" to "tight". Working points that are "looser" tend to have a high efficiency for accepting
real objects, but perhaps a poor rejection rate for "fake" objects. Working points that are
"tighter" tend to have lower efficiencies for accepting real objects, but much better rejection
rates for "fake" objects. The choice of working point is highly analysis dependent! Some analyses
value efficiency over background rejection, and some analyses are the opposite.

The "standard" identification and isolation algorithm results can be accessed from the physics
object classes. CMS maintains reference pages that document the various options:

 * General: [CMS Physics Objects 2011](http://opendata.cern.ch/docs/cms-physics-objects-2011)
 * Muons: [SWGuide Muon ID](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMuonId)
 * Electrons: [EGamma Public Data](https://twiki.cern.ch/twiki/bin/view/CMSPublic/EgammaPublicData)
 * Photons: [EGamma Public Data](https://twiki.cern.ch/twiki/bin/view/CMSPublic/EgammaPublicData), [Summary document (see Table 1 for photon ID)](https://cms-physics.web.cern.ch/cms-physics/public/EGM-10-006-pas.pdf)
 * Tau leptons: [Legacy Tau ID Run 1](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookPFTauTagging#Legacy_Tau_ID_Run_I), [Nutshell Recipe](https://twiki.cern.ch/twiki/bin/view/CMSPublic/NutShellRecipeFor5312AndNewer)

## Muons

The CMS Muon object group has created member functions for the identification algorithms that simply
storing pass/fail decisions about the quality of each muon. As shown below, the algorithm depends
on which vertex is being considered as the primary interaction vertex!

Hard processes produce large angles between the final state partons. The final object of interest will be separated from 
the other objects in the event or be "isolated". For instance, an isolated muon might be produced in the decay of a W boson.
In contrast, a non-isolated muon can come from a weak decay inside a jet. 

Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from
neutral hadrons, and energy from photons, all in a cone of radius $\Delta R < 0.3$ or 0.4 around
the muon. Many algorithms also feature a "correction factor" that subtracts average energy expected
from pileup contributions to this cone. Decisions are made by comparing this energy sum to the
transverse momentum of the muon. 

~~~
for (auto it = muons->begin(); it != muons->end(); it++) {

    // If this muon has isolation quantities...
    if (it->isPFMuon() && it->isPFIsolationValid()) {

       // get the isolation info in a certain cone size:
       auto iso04 = it->pfIsolationR04();

       // and calculate the energy relative to the muon's transverse momentum
       value_mu_pfreliso04all[value_mu_n] = (iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/it->pt();
    }

    // Store the pass/fail decisions about Tight ID
    value_mu_tightid[value_mu_n] = muon::isTightMuon(*it, *vertices->begin());
}
~~~
{: .source}

>## Challenge: alternate IDs and isolations
>
>Using the documentation on the TWiki page, adjust the 0.4-cone muon isolation calculation
>to apply the "DeltaBeta" pileup correction.
>Also add the pass/fail information about the Loose and Soft identification working points.
{: .challenge}

## Tau leptons

The CMS Tau object group relies almost entirely on pre-computed algorithms to determine the
quality of the tau reconstruction and the decay type. Since this object is not stable and has
several decay modes, different combinations of identification and isolation algorithms are
used across different analyses. The TWiki page provides a large table of available algorithms.

In contrast to the muon object, tau algorithm results are typically saved in the AOD files
as their own PFTauDisciminator collections, rather than as part of the tau object class.

~~~
// Get the tau collection
Handle<PFTauCollection> taus;
iEvent.getByLabel(InputTag("hpsPFTauProducer"), taus);

// Get various tau discriminator collections
Handle<PFTauDiscriminator> tausLooseIso, tausVLooseIso, tausMediumIso, tausTightIso,
                           tausDecayMode, tausRawIso;

iEvent.getByLabel(InputTag("hpsPFTauDiscriminationByDecayModeFinding"),tausDecayMode);
iEvent.getByLabel(InputTag("hpsPFTauDiscriminationByRawCombinedIsolationDBSumPtCorr"),tausRawIso);
iEvent.getByLabel(InputTag("hpsPFTauDiscriminationByVLooseCombinedIsolationDBSumPtCorr"),tausVLooseIso);
//...etc...
~~~
{: .source}

The tau discriminator collections act as pairs, containing the index of the tau and the value
of the discriminant for that tau. Note that the arrays are filled by calls to the individual
discriminant objects, but referencing the vector index of the tau in the main tau collection.

~~~
for (auto it = taus->begin(); it != taus->end(); it++) {

    // store the tau decay mode
    value_tau_decaymode[value_tau_n] = it->decayMode();

    // Discriminators
    const auto idx = it - taus->begin();
    value_tau_iddecaymode[value_tau_n] = tausDecayMode->operator[](idx).second;
    value_tau_idisoraw[value_tau_n] = tausRawIso->operator[](idx).second;
    value_tau_idisovloose[value_tau_n] = tausVLooseIso->operator[](idx).second;
    // ...etc...
}
~~~
{: .source}

>## Challenge: alternate tau IDs
>
>Many other tau discriminants exist. Based on information from the TWiki, 
>save the values for some discriminants that are based on rejecting electrons or muons.
{: .challenge}

## Detector-related information

Most `reco::<object>` classes contain member functions that return detector-related information. In the
case of electrons and photons, we see this information used as identification criteria:

~~~
value_el_isLoose[value_el_n] = false;
value_el_isMedium[value_el_n] = false;
value_el_isTight[value_el_n] = false;
if ( abs(it->eta()) <= 1.479 ) {
  if ( abs(it->deltaEtaSuperClusterTrackAtVtx())<.007 && abs(it->deltaPhiSuperClusterTrackAtVtx())<.15 &&
       it->sigmaIetaIeta()<.01 && it->hadronicOverEm()<.12 &&
       abs(trk->dxy(pv))<.02 && abs(trk->dz(pv))<.2 &&
       missing_hits<=1 && pfIso<.15 && passelectronveto==true &&
       abs(1/it->ecalEnergy()-1/(it->ecalEnergy()/it->eSuperClusterOverP()))<.05 ){

    value_el_isLoose[value_el_n] = true;

    if ( abs(it->deltaEtaSuperClusterTrackAtVtx())<.004 && abs(it->deltaPhiSuperClusterTrackAtVtx())<.06 && abs(trk->dz(pv))<.1 ){
      value_el_isMedium[value_el_n] = true;

      if (abs(it->deltaPhiSuperClusterTrackAtVtx())<.03 && missing_hits<=0 && pfIso<.10 ){
        value_el_isTight[value_el_n] = true;
      }
    }
  }
}
~~~
{: .source}

The first two criteria (`deltaEta` and `deltaPh`) indicate how the electron's trajectory varies between the track and the ECAL cluster,
with smaller variations preferred for the "tightest" quality levels. The `sigmaIetaIeta` criterion describes the variance of the ECAL
cluster in psuedorapidity (recall the "ieta" labels on the red LEGO bricks!). There are futher criteria for the ratio of hadronic to 
electromagnetic energy deposits, the track impact parameters, and the difference between the ECAL energy and electron's momentum --
all of which are expected to be small for genuine electrons with well-reconstructed tracks. Finally, a good electron should have very 
few "missing hits" (gaps in the trajectory through the inner tracker), be reasonably isolated from other particle-flow candidates in the
nearby spatial region, and should pass an algorithm that rejects electrons from photon conversion in the tracker. Similar information from 
the detector is used to form the identification criteria for all physics objects. 


>## Challenge: muon detector information
>
>Using the documentation links provided above, familiarize yourself with the detector-related information used to 
>construct muon identification criteria. If time permits, attempt to replicate the content of the existing "soft" or "tight" 
>ID flags using a combination of cuts on these quantities.
{: .challenge}




{% include links.md %}

