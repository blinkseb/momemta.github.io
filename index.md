---
layout: default
title: {{ site.name }}
---

# The MoMEMta project

**MoMEMta** is a software package designed in a modular way to cover the needs of the experimental analysis workflows at the LHC. MoMEMta provides working examples for the most common final states (ttbar, WW, ...). If you are expert user, be prepared to feel the freedom of configure your MEM computation at all levels.

MoMEMta is based on:

 - C++, ROOT, Lua scripting language
 - Cuba library for integration
 - External PDFs (LHAPDF by default)
 - External Matrix Element (provided with MadGraph C++ exporter)

# Getting started

You first need to install MoMEMta. A guide on how-to install from source is available [on our github repository](https://github.com/MoMEMta/MoMEMta/blob/prototype/README.md#install). You can then browse our [documentation](http://momemta.github.io/MoMEMta/) to learn more about MoMEMta.

# Usage example

```cpp
#include <momemta/ConfigurationReader.h>
#include <momemta/Logging.h>
#include <momemta/MoMEMta.h>
#include <momemta/Utils.h>

int main(int argc, char** argv) {

    ConfigurationReader configuration("tt_fullyleptonic.lua");
    MoMEMta weight(configuration.freeze());

    LorentzVector p3(16.171895980835, -13.7919054031372, -3.42997527122497, 21.5293197631836);
    LorentzVector p4(-55.7908325195313, -111.59294128418, -122.144721984863, 174.66259765625);
    LorentzVector p5(-18.9018573760986, 10.0896110534668, -0.602926552295686, 21.4346446990967);
    LorentzVector p6(71.3899612426758, 96.0094833374023, -77.2513122558594, 142.492813110352);

    std::vector<std::pair<double, double>> weights = weight.computeWeights({p3, p4, p5, p6});

    std::cout << "Result: " << weights.back().first << " +- " << weights.back().second;
    
    return 0;
}
```

More examples can be found [here](https://github.com/MoMEMta/MoMEMta/tree/prototype/examples).

# Our team

 - Sébastien Brochet
 - Alessia Saggio
 - Miguel Vidal
 - Sébastien Wertz

# Support and contact

Having trouble with MoMEMta? [Email us](mailto:momemta-contact@cern.ch)

---

## *Acknowledgments*

*This work would not have been possible without the help of the [MadWeight](https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/MadWeight) team*
