---
layout: post
title: Algorithms I
categories: cs 
description: cs, algorithms
---

<!-- unfoldr is the second program of my complex systems software suite that I open-source. ([histogramr]({% post_url 2015-03-08-histogramr %}) was the first one.) For those of you who study the spectral features of complex systems such as random networks I think unfoldr will prove quite useful. Given the eigenvalues of an ensemble of random matrices, unfoldr calculates the nearest-neighbor level spacings of the unfolded spectrum, either as a whole or for *slices* of it. You can specify how you want to cut the spectrum into slices — linearly, logarithmically —, and unfoldr will calculate the level spacings for each slice individually. With this you can study how the level spacing statistics change with energy and whether or not there is a phase transition in the spectrum. <span class="more"></span> I have used this technique to pinpoint the localization transition in the energy spectrum of an ultra-cold Rydberg gas, {% cite scholak2014spectral --file 2015-05-06-unfoldr %}. I comment on the purpose and the scientific value of the nearest-neighbor level spacings statistics [below](#theory).

Like histogramr, unfoldr reads and writes [HDF5](http://www.hdfgroup.org/HDF5/) files. HDF5 files are organized in a hierarchical, file-system-like data structure. HDF5 is [commonly used in scientific environments](https://www.hdfgroup.org/users.html). It is designed to handle big scientific data sets. Please see my [earlier blog post]({% post_url 2015-03-08-histogramr %}) for a description of HDF5 and how it is used.

unfoldr and histogramr are designed to work in tandem. Output from unfoldr can (but doesn't have to) be processed by histogramr. unfoldr produces a HDF5 file that contains the level spacings in a single data set:

```bash
＄ unfoldr -d "spectrum" -m "eigval" -o nnls.h5 spectra.h5
```

And histogramr then creates a discretized probability density function from it:

```bash
＄ histogramr -d "inf" -m "spacing" -b .1 -l 0,10 -o nnlsd.h5 nnls.h5
```

Boom! You've got the nearest-neighbor level spacing distribution stored nicely in `nnlsd.h5`. Then you can do this:

```python
import numpy as np
import h5py
import plotly.plotly as py
from plotly.graph_objs import *

fS_Poisson = np.array((lambda x: [x, np.exp(-x)])(np.linspace(0, 4, 100))).transpose()
fS_GOE = np.array((lambda x: [x, np.pi*x*np.exp(-np.pi*x**2./4.)/2.])(np.linspace(0, 4, 100))).transpose()

h5file = h5py.File('nnlsd.h5','r')
fS_unfoldr = np.array(h5file['probability density'])
h5file.close()

trace_Poisson = Scatter(x=fS_Poisson[:,0],
                        y=fS_Poisson[:,1],
                        mode='lines',
                        name=u"Poisson distribution")
trace_GOE = Scatter(x=fS_GOE[:,0],
                    y=fS_GOE[:,1],
                    mode='lines',
                    name=u"Wigner's surmise")
trace_unfoldr = Scatter(x=fS_unfoldr[:,0],
                        y=fS_unfoldr[:,1],
                        mode='markers',
                        name=u'unfoldr data',
                        marker=Marker(symbol='square'))
data = Data([trace_Poisson, trace_GOE, trace_unfoldr])
layout = Layout(title=u'Nearest-neighbor level spacing distribution')
fig = Figure(data=data, layout=layout)

py.iplot(fig, filename="unfoldr-demonstration")
```

The output is:

<div>
  <a href="https://plot.ly/~tscholak/26/" target="_blank" title="Nearest-neighbor level spacing distribution" style="display: block; text-align: center;"><img src="https://plot.ly/~tscholak/26.png" alt="Nearest-neighbor level spacing distribution" style="max-width: 100%;"  onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
  <script data-plotly="tscholak:26" src="https://plot.ly/embed.js" async></script>
</div>

It looks like the data (squares) agrees with [Wigner's surmise](#Wigners-surmise)!

# Get unfoldr today from GitHub!

You can [download unfoldr from GitHub](https://github.com/tscholak/unfoldr). In Linux or Mac OS X, open a terminal session and run:

```bash
＄ git clone git@github.com:tscholak/unfoldr.git
```

(get Git [here](http://git-scm.com)). Since I released unfoldr under the GPLv3, you can use, study, share, and modify it for *free* (as long as you retain these rights for others). unfoldr is written in [Python](https://www.python.org) and uses the [setuptools](https://pypi.python.org/pypi/setuptools). From within the source code directory, run:

```bash
＄ python setup.py install
```

Make sure your Python's `bin` directory is in your `$PATH`. On Mac OS X, if you use Macports and haven't done so already, add

```bash
＄ export PATH=/opt/local/Library/Frameworks/Python.framework/Versions/2.7/bin:$PATH
```

to your `~/.bash_login` or `~/.bash_profile` file. Once done, you can call unfoldr from the command line by invoking `unfoldr`. `unfoldr -h` will show you the command line arguments.

# <a name="theory"></a>What exactly is unfoldr good for?

To understand the use cases of unfoldr, I first have to review some complex system theory. If you know all that stuff, skip [ahead](#nearest-neighbor-level-spacing-statistics).

Suppose you are dealing with a complex system, i.e. a large, complicated, but deterministic system or a system whose complexity comes from some intrinsic randomness. Suppose further that you do not seek a full and exact theory describing all phenomena that could possibly manifest in your complex system. Maybe you tried to derive such a theory, but you realized the impossibility of such a task. In fact, your resentment of the task has made you become ignorant of all the details of the system that were once so precious to you. Believe it or not, this is not such a bad thing. There is a way out of your dilemma, and it involves replacing your model — the Hamiltonian, the weighted adjacency matrix, Laplacian matrix, or whatever description of your system you have — by a *random matrix* (or an ensemble of random matrices) that has the same invariances and symmetry properties as your system. As radical as it may sound, in most cases, it works surprisingly well. The theory that makes all this possible is called *random matrix theory* (RMT) {% cite Wigner1967Random-Matrices Dyson1970Correlations-be Mehta1991Random-matrices Guhr1998Random-matrix-t Stockmann1999Quantum-chaos --file 2015-05-06-unfoldr %}.

## Random matrix theory and Gaussian ensembles

RMT is concerned with the statistical properties (particularly, of eigenvalues and eigenvectors) of large $N \times N$ matrices $M$ with random elements $M\_{i j}$. Within RMT, all results are derived from the probability density function $f\_{M}$ of $M$. RMT is relevant in math {% cite Tao2012Topics-in-Rando --file 2015-05-06-unfoldr %}, theoretical physics (think complex nuclei, quantum and microwave billiards, metals with randomly distributed impurities, Rydberg gases {% cite scholak2014spectral --file 2015-05-06-unfoldr %}, Boson sampling {% cite Walschaers2014A-Statistical-B --file 2015-05-06-unfoldr %}, ...), finance {% cite Bouchaud2009Financial-Appli --file 2015-05-06-unfoldr %}, and the study of networks and communities (social networks, computer networks, etc.). In physics, for instance, $M$ can be the matrix representation of a realization of the system's Hamiltonian.

Of course, for all interesting complex systems, the density $f\_{M}$ is highly nontrivial, impossible to obtain from first principles, and thus simply not known. The first step in an RMT treatment of a complex system is therefore to attempt to simplify or to guesstimate $f\_{M}$. As mentioned already above, these attempts are guided by the invariance properties of the complex system. There are three seminal random matrix ensembles that result from such considerations:

- the Gaussian orthogonal ensemble (GOE),
- the Gaussian unitary ensemble (GUE) and
- the Gaussian symplectic ensemble (GSE).

As the names suggest, the densities $f\_{M}$ of these ensembles respect the invariance with respect to orthogonal, unitary, or symplectic similarity transformations, respectively... ;) In other words, the core assumption for these ensembles is that all real (complex, quaternion) basis sets are equally well suited to describe the system. Furthermore, all matrix elements $M\_{i j}$ are independent, and the joint probability densities $f\_{M}$ decompose into products of their marginals, $f\_{M} = f\_{M\_{1,1}} f\_{M\_{1,2}} \cdots f\_{M\_{N,N}}$. The marginals $f\_{M\_{i j}}$ are normally distributed with zero mean and fixed variance. For the GOE, they read
\\[
  \begin{align\*}
    f\_{M\_{i i}}(m) & = \frac{1}{\sqrt{2 \pi \sigma\^2}} \, \mathrm{e}\^{- \frac{m}{2 \sigma\^2}}, \\\\
    f\_{M\_{i j}}(m) & = \frac{1}{\sqrt{\pi \sigma\^2}} \, \mathrm{e}\^{- \frac{m}{\sigma\^2}},
  \end{align\*}
\\]
where $\sigma\^2$ is the variance of the diagonal elements $M\_{i i}$. 

The three Gaussian random matrix ensembles have been studied ad nauseam. The eigenvalue density is known (it's the [Wigner semi-circle](http://en.wikipedia.org/wiki/Wigner_semicircle_distribution)), the eigenstate localization properties are known (they are extended {% cite Erdos2007Semicircle-law- --file 2015-05-06-unfoldr %}, i.e. every eigenvector extends over a fraction of the system that scales as the system size $N$), the level-spacing distribution is known (it's Wigner's surmise, see [below](#Wigners-surmise)), etc. The importance of the Gaussian ensembles stems from the fact that they are *universal*. If a closed system's behavior is chaotic in the classical limit, then, statistically speaking, its corresponding wave or quantum behavior coincides with one of the Gaussian ensembles {% cite Mehta1991Random-matrices --file 2015-05-06-unfoldr %}. Integrable (non-chaotic) systems, on the other hand, have a completely different statistical fingerprint. If your complex system turns out to coincide with universal Gaussian statistics, then you are done. There is nothing left to solve or discover, because everything you possibly would like to know (and can be known) about your system has already been discovered. And if your complex system turns out to be different, then there is still a good chance that its statistics are described by one of the other important random matrix ensembles:

- the circular random matrices {% cite Guhr1998Random-matrix-t Stockmann1999Quantum-chaos --file 2015-05-06-unfoldr %},
- the stable random matrices {% cite Cizeau1994Theory-of-Levy- --file 2015-05-06-unfoldr %},
- the Euclidean random matrices {% cite Goetschy2013Euclidean-rando --file 2015-05-06-unfoldr %},
- the power-law banded random matrices {% cite Mirlin1996Transition-from --file 2015-05-06-unfoldr %},
- the Laplacians of Erdős–Rényi (Poissonian) random graphs {% cite Bollobas2001Random-Graphs --file 2015-05-06-unfoldr %},

you name it.

## <a name="nearest-neighbor-level-spacing-statistics"></a>Eigenvalue correlations and level repulsion

At this point, you may wonder: How can I find out about the statistics of my complex system? How do I get its statistical fingerprint? How can I compare my system with the random matrix ensembles? unfoldr helps you with that. It turns out you can learn a lot from the data:

One of the most useful statistical metrics for random matrix ensembles is the nearest-neighbor level spacing (NNLS) density, denoted here and in the following by $f\_{S}$. What does $f\_{S}$ tell you? Formally,
\\[
  f\_{S}(s) \, \mathrm{d}s = \overline{\delta(s - S_{\nu})} \, \mathrm{d}s
\\]
states the ensemble averaged probability to sample two adjacent eigenvalues (also called eigenenergies) $\Lambda\_{\nu}$, $\Lambda\_{\nu+1}$ that are separated by an energy difference $S\_{\nu} = \Lambda\_{\nu+1} - \Lambda\_{\nu}$ between $s$ and $s + \mathrm{d}s$. Defining this makes sense as long as you sort your eigenvalues $\Lambda\_{\nu}$ in ascending order for each realization of $M$, that is $\Lambda\_{1} \le \Lambda\_{2} \le \ldots \le \Lambda\_{N}$.

The NNLS density is important, because its shape depends on the correlations and interactions between *all* eigenvalues of the random matrix. It is a fingerprint of the eigenvalue correlations. If you have the eigenvalues (or eigenenergies) of your system, unfoldr can compute the level spacings for you. Afterwards, you can use histogramr to get the NNLS density and thus the correlation fingerprint of *your* system. That fingerprint can then be compared to the fingerprints of, e.g., the universal Gaussian matrices.

The eigenvalue correlations have a deterministic system-dependent part and a random universal part. It is important to note that only the latter part may be used for comparisons between different systems or matrix ensembles. To extract the universal part, the spectrum of your complex system must be *unfolded*. Spectral unfolding is a transformation that maps the ensemble averaged level spacing,
\\[
  \overline{S} = \int\_{0}\^{\infty} s \, f\_{S} (s) \, \mathrm{d}s,
\\]
to a constant, i.e. $\overline{S} = 1$. unfoldr will do that for you (hence the name). So you don't have to worry about it.

If there are no dependencies between the eigenvalues, all distances between them will be uncorrelated, and $f\_{S}$ will thus be identical to the Poisson distribution,
\\[
  f\^{\mathrm{P}}\_{S} = \mathrm{e}\^{-s}.
\\]
<a name="Wigners-surmise"></a>By contrast, the spectra of the universal Gaussian ensembles (GOE, GUE, GSE) are correlated. Their level-spacing statistics are not described by $f\^{\mathrm{P}}\_{S}$. For instance, the NNLS statistics of the GOE are well described by Wigner's surmise, i.e. by the Rayleigh distribution
\\[
  f\^{\mathrm{GOE}}\_{S}(s) = \frac{\pi}{2} \, s \, \mathrm{e}\^{-\frac{\pi}{4} s\^2}.
\\]
The densities of the GUE and GSE, on the other hand, follow
\\[
  \begin{align\*}
    f\^{\mathrm{GUE}}\_{S}(s) & = \frac{32}{\pi\^2} \, s\^2 \, \mathrm{e}\^{-\frac{4}{\pi} s\^2}, \\\\
    f\^{\mathrm{GSE}}\_{S}(s) & = \frac{2\^{18}}{3\^6 \pi\^3} \, s\^4 \, \mathrm{e}\^{-\frac{64}{9 \pi} s\^2},
  \end{align\*}
\\]
respectively, to very good approximation (especially when $N$ is large). The correlations between the eigenvalues is manifested in the frequency of occurrence of small energy differences $s$. A negative deviation from the Poissonian statistics $f\^{\mathrm{P}}\_{S}$ signifies *level repulsion* (which coincidentally indicates eigenvector delocalization {% cite Izrailev1990Simple-models-o --file 2015-05-06-unfoldr %}). When we look at Wigner's surmise $f\^{\mathrm{GOE}}\_{S}(s)$, we see that it increases linearly for small spacings $s$. In other words, the probability of sampling two neighboring eigenvalues that are separated by a small distance $S$ between $s$ and $s + \mathrm{d}s$ is *increasing* with $s$. Not only do eigenvalues not cluster, they avoid each other! The repulsion is even stronger for the other universal ensembles, since $f\^{\mathrm{GUE}}\_{S}(s) \sim s\^2$ and $f\^{\mathrm{GSE}}\_{S}(s) \sim s\^4$ for small $s$.

Let's assume now you have calculated the NNLS density using unfoldr and histogramr and you are staring at the result. What can you conclude?

- If it looks like the Poisson distribution $f\^{\mathrm{P}}\_{S}$, then you know that your eigenvalues are uncorrelated. It is also quite likely that your eigenstates are localized, either strongly or weakly {% cite Izrailev1990Simple-models-o --file 2015-05-06-unfoldr %}.
- If it looks like Wigner's surmise, like $f\^{\mathrm{GUE}}\_{S}$, or like $f\^{\mathrm{GSE}}\_{S}$, then you know that your system agrees with universal Gaussian statistics. This is a big deal and can help you with a lot of things. (In physics, everything nice is Gaussian.) You also know now that your eigenvectors are delocalized.
- If it looks like a mix between Poisson and universal Gaussian statistics, then you may have a blend between two or more statistically independent subspectra that correspond to either statistics. You can check that by estimating $\lim_{s \to 0} f\_{S}(s)$. If this limit is finite, then this will be consistent with such a blend {% cite Berry1984Semiclassical-l --file 2015-05-06-unfoldr %}. However, if your $f\_{S}(s) \sim s\^\alpha$ for small $s$, then you have most likely interaction between the subspectra {% cite Backer2011Fractional-Powe --file 2015-05-06-unfoldr %}. In other words, your subspectra are not independent and their blend results in something that is different from either of them. You may have a localization transition in your system.

If you think you have a localization transition in your system (or maybe just a transition between Poisson and Wigner-Dyson statistics under variation of the energy), you can get further insight into this by letting unfoldr calculate the NNLSs for individual slices through the spectral data. For instance,

```bash
＄ unfoldr -d "spectrum" -m "eigval" -l -10,10 -b .5 -o nnls.h5 spectra.h5
＄ histogramr -d "-9.75" -m "spacing" -b .1 -l 0,10 -o nnlsd_-9.75.h5 nnls.h5
＄ histogramr -d "-9.25" -m "spacing" -b .1 -l 0,10 -o nnlsd_-9.25.h5 nnls.h5
...
＄ histogramr -d "9.75" -m "spacing" -b .1 -l 0,10 -o nnlsd_9.75.h5 nnls.h5
```

calculates not one, but twenty NNLS densities that are based exclusively on the spectral data in the intervals $[-10, -9.5)$, $[-9.5,-9)$, $\ldots$, $[9.5,10)$. With this data you can easily follow the change of the NNLS statistics with the energy {% cite scholak2014spectral --file 2015-05-06-unfoldr %}.

## Unfolding — theory and numerical implementation

Below I first explain how unfolding works {% cite Guhr1998Random-matrix-t --file 2015-05-06-unfoldr %} and then how it is [implemented in unfoldr](#implementation). As an auxiliary tool, I first define the spectral staircase function
\\[
  \mathcal{N}(\lambda) = \sum\_{\nu = 1}\^N \Theta\left(\lambda - \Lambda\_{\nu}\right) = \\#\left\\{\nu \middle| \Lambda\_{\nu} < \lambda\right\\},
\\]
where $\Theta(\lambda) = \int\_\infty\^\lambda \delta(\lambda') \, \mathrm{d}\lambda'$ is the Heaviside step function. $\mathcal{N}(\lambda)$ is the number of eigenvalues $\Lambda\_{\nu}$ that are smaller than $\lambda$. We separate $\mathcal{N}(E)$ into a smooth part,
\\[
  \overline{\mathcal{N}}(\lambda) = N F\_{\Lambda}(\lambda) = N \int\_{-\infty}\^{\lambda} f\_{\Lambda}(\lambda') \, \mathrm{d}\lambda',
\\]
and a fluctuating part,
\\[
  \mathcal{N}\_{\mathrm{osc}} = \mathcal{N}(\lambda) - \overline{\mathcal{N}}(\lambda),
\\]
i.e. into the ensemble average given by the cumulative energy level distribution function $F\_{\Lambda}$ and the difference between the realization's value of $\mathcal{N}(\lambda)$ and the ensemble average, respectively. $f\_{\Lambda}$ is the density of states, defined as
\\[
  f\_{\Lambda}(\lambda) = \frac{1}{N} \, \overline{\operatorname{Tr} \delta \left(\lambda - M\right)}.
\\]
$f\_{\Lambda}(\lambda) \, \mathrm{d}\lambda$ is the ensemble averaged probability to find an eigenvalue $\Lambda$ of $M$ in the interval $\left[\lambda, \lambda + \mathrm{d}\lambda\right]$.

Now, unfolding is defined as the mapping
\\[
  \Lambda\_{\nu} \mapsto \tilde{\Lambda}\_{\nu} = \overline{\mathcal{N}}\left(\Lambda\_{\nu}\right).
\\]
This is done for all eigenvalues $\{\Lambda\_{\nu}\}$ of a given realization. Thus, by construction, the ensemble averaged spectral staircase function of the unfolded spectrum $\left\\{\tilde{\Lambda}\_{\nu}\right\\}$ evaluated for the scaled energy $\lambda$ is equal to $\lambda$ itself,
\\[
  \overline{\tilde{\mathcal{N}}}(\lambda) = \overline{\\#\left\\{\nu \middle| \tilde{\Lambda}\_{\nu} < \lambda\right\\}} = \lambda,
\\]
and the corresponding unfolded mean level spacing equals one, $\overline{\tilde{S}} = 1$, such that the level spacing density $f\_{\tilde{S}}$ of the unfolded eigenvalues is normalized with respect to
\\[
  \begin{align\*}
    \int\_0\^\infty f\_{\tilde{S}}(s) \, \mathrm{d}s & = 1, \\\\
    \int\_0\^\infty s \, f\_{\tilde{S}}(s) \, \mathrm{d}s & = \overline{\tilde{S}} = 1.
  \end{align\*}
\\]
<a name="implementation"></a>How is unfolding done in unfoldr? When you load your spectral data with unfoldr, two-thirds of them are taken away to populate what I call the unfolding pool. The unfolding pool is a flat, sorted array of eigenvalues. I use it to estimate the ensemble averaged spectral staircase function $\overline{\mathcal{N}}$ and to do the transformation $\Lambda\_{\nu} \mapsto \tilde{\Lambda}\_{\nu}$. The following Python pseudo code shows how this is implemented:

```c
i = 0
for j in range(number of eigenvalues):
  while unfolding_pool[i] < eigenvalue[j]:
    i += 1
    if i >= length of the unfolding pool:
      break
  if j + 1 < number of eigenvalues:
    level_spacing[j] -= i / (number of spectra in the unfolding pool)
  if j > 0:
    level_spacing[j-1] += i / (number of spectra in the unfolding pool)
```

This calculation is done for each of the remaining spectra (the third that doesn't go into the unfolding pool). The above piece of code does not only unfold the spectrum, but also calculates the level spacings. The code is compiled with Cython, so it should be fast.

You can control the size of the unfolding pool with the command line argument `-p`. The default value is 2. That means that the number of spectra in the unfolding pool is twice as large as the number of spectra that are unfolded. The default value should work in most cases. However, if you see oscillations or weird artifacts in your level spacing densities, then try a larger number.

* * *

# References

{% bibliography --file 2015-05-06-unfoldr --cited %} -->



