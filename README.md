# PyPALM
Flexible methods for permutation analysis of linear models. 

## Summary
PyPALM provides functions for testing the significance of contrasts in multiple linear regression models using non-parametric permutation testing. ***The major advantage of PyPALM over existing packages is the ability to specify a custom function for shuffling data or generating surrogate data sets.*** This is useful in situations where data have limited exchangability, such as in the presence of autocorrelation (e.g. when comparing two brain maps).

## What it does and how it works
For a linear model of the form: Y ~ X + Z.
- Y is the response variable.
- X is one or more predictor variables of interest.
- Z is one or more predictor variables of non-interest.

PyPALM currently provides two methods for estimating the significance of the effect of X while accounting for ("controllilng") the effect of Z:
1. **Freedman-Lane:** Permute the residulas of a reduced model (Y ~ Z).
2. **Manly:** Permute Y.

The Freedman-Lane method is generally recommended as it preserves the relationships between Z and the other variables while providing good power and control over error rates (Winkler et al., 2014). The Manly method is provided for situations where complex dependencies between or within variables preclude application of the Freedman-Lane approach. 

## Using a custom permutation function.
The key feature of PyPALM is the ability for users to provide a custom function for permuting data (or generating surrogate data). Below is a short example of how to do this:

```python
# Define a function which only shuffles values within N sub-blocks of the data.
def split_shuffle(data, n_blocks=2):
    import numpy as np
    blocks = np.split(data, n_blocks)
    shuffled_data = []
    for block in blocks:
        shuffled_data.append(np.random.permutation(block))
    shuffled_data = np.concatenate(shuffled_data, axis=None)

    return shuffled_data

# Define a helper function to wrap split_shuffle when used by PyPALM
def perm_helper(data, n_perms, n_blocks):
    surrogates = []
    for i in range(n_perms)
        surrogates.append(split_shuffle(data, n_blocks))

    return surrogates

# Create a dictionary to feed arguments into split_shuffle
perm_args = {'n_blocks':15}

# Run PyPALM using the Freedman-Lane method and your custom permutation function
import pypalm

t_val, p_vals, model, null_dist, surrogates = pypalm.freedman_lane(data_df, 'Yvar', 'Xvar', 'Zvar', stat='tstat', n_perms=10000, perm_func=perm_helper, perm_func_args=perm_args, return_surrogates=True, return_null=True)
```
If no custom function is provided, PyPALM will use random shuffling. 

## PyPALM is a work in progress
I can make no guarantees that it is suitable for use by anyone other than myself, and in the specific cases it was designed for. That said, I have tested it against the [nptest](https://cran.r-project.org/web/packages/nptest/index.html) R package and found it to produce p-values that are identical up to at least two decimal places. See the `test` directory for details.

## Background and Acknowledgements
PyPALM is based heavily (in concept, though not necessarily in code) on [FSL PALM](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/PALM) and related work by [Anderson Winkler](https://github.com/andersonwinkler). That said, please note that PyPALM has no official relationship with the official PALM package, nor does it come anywhere close (yet?) to fully implementing all the features of PALM.

If you use PyPALM, in addition to citing this repository, please also cite the key publication on which this work is based:

Winkler AM, Ridgway GR, Webster MA, Smith SM, Nichols TE. [Permutation inference for the general linear model.](https://doi.org/10.1016/j.neuroimage.2014.01.060) NeuroImage, 2014;92:381-397

## Further Reading
Freedman, D., & Lane, D. (1983). A Nonstochastic Interpretation of Reported Significance Levels. Journal of Business & Economic Statistics: A Publication of the American Statistical Association, 1(4), 292–298. https://doi.org/10.1080/07350015.1983.10509354

Manly, B.F.J. (1986) Randomization and regression methods for testing for associations with geographical, environmental and biological distances between populations. Res Popul Ecol 28, 201–218. https://doi.org/10.1007/BF02515450

Anderson, M. J. (2001). Permutation tests for univariate or multivariate analysis of variance and regression. Canadian Journal of Fisheries and Aquatic Sciences. Journal Canadien Des Sciences Halieutiques et Aquatiques, 58(3), 626–639. https://doi.org/10.1139/f01-004

