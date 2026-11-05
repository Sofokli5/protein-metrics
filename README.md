# protein-metrics
What do proteins have to tell us about themselves?

A Google Colab notebook that analyses **AlphaFold 3** models of an enzyme–reductase complex and measures how a bound drug changes it.

## What it does

For each drug, a model of the complex with the drug is compared with a model without it. The notebook evaluates:

- **Model confidence:** how reliable the predicted structures and interfaces are.
- **Interface size:** the surface buried between the two proteins, and how buried the drug is.
- **Structural change:** how the reductase moves or changes shape when the drug is present.
- **Drug position and binding:** where the drug sits and how favourably it binds.
- **Protein–protein affinity:** the predicted strength of the enzyme–reductase interaction.

The results for each drug are collected in one table. Drugs can then be filtered for reliability, grouped by binding site and ranked by their effect on the complex.

## Tools

AlphaFold 3 (input models), MDAnalysis, Biopython, FreeSASA, AutoDock Vina, OpenBabel, PRODIGY, pandas.

## Note

All values are predictions from computational models. They are meant for comparisons between drugs, not as absolute measurements.

## License

MIT
