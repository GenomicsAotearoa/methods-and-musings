### Evaluation of MAGs

----

#### Assembled contigs

To begin, we'll get a quick overview of how many contigs (total and after length filtering) were actually obtained from each assembler/error correction permutation.

<table border='1'>
	<tr>
		<th></th><th></th><th colspan='3'>Contigs</th><th colspan='3'>Contigs (5 kbp+)</th>
	</tr>
	<tr>
		<th>Data type</th><th>Assembler</th><th>S4R3</th><th>S5R2</th><th>S6R1</th><th>S4R3</th><th>S5R2</th><th>S6R1</th>
	</tr>
	<tr>
		<td>Uncorrected</td><td>CANU</td>
		<td>23,160</td><td>22,862</td><td>11,442</td><td>10,692</td><td>13,715</td><td>???</td>
	</tr>
		<tr>
		<td></td><td>Flye</td>
		<td>3,172</td><td>3,487</td><td>1,442</td><td>3,143</td><td>3,475</td><td>1,436</td>
	</tr>
		<tr>
		<td></td><td>MetaFlye</td>
		<td>3,255</td><td>3,722</td><td>1,754</td><td>3,231</td><td>3,716</td><td>1,743</td>
	</tr>
		<tr>
		<td></td><td>MetaFlye (tuned)</td>
		<td>3,256</td><td>3,718</td><td>1,754</td><td>3,233</td><td>3,712</td><td>1,743</td>
	</tr>
		<tr>
		<td></td><td>MiniASM</td>
		<td>716</td><td>1,047</td><td>412</td><td>695</td><td>1,029</td><td>405</td>
	</tr>
		<tr>
		<td></td><td>Shasta</td>
		<td>337,785</td><td>483,755</td><td>223,642</td><td>330,742</td><td>467,862</td><td>220,336</td>
	</tr>
	<tr>
		<td>FMLRC</td><td>CANU</td>
		<td>4,781</td><td>5,151</td><td>-</td><td>2,191</td><td>2,572</td><td>-</td>
	</tr>
		<tr>
		<td></td><td>Flye</td>
		<td>2,092</td><td>1,448</td><td>1,057</td><td>2,082</td><td>1,444</td><td>1,055</td>
	</tr>
		<tr>
		<td></td><td>MetaFlye</td>
		<td>2,363</td><td>1,864</td><td>1,411</td><td>2,357</td><td>1,859</td><td>1,410</td>
	</tr>
		<tr>
		<td></td><td>MetaFlye (tuned)</td>
		<td>2,364</td><td>1,863</td><td>1,408</td><td>2,359</td><td>1,858</td><td>1,407</td>
	</tr>
		<tr>
		<td></td><td>MiniASM</td>
		<td>740</td><td>1,326</td><td>728</td><td>719</td><td>1,306</td><td>423</td>
	</tr>
		<tr>
		<td></td><td>Shasta</td>
		<td>330,295</td><td>468,239</td><td>219,591</td><td>325,307</td><td>458,307</td><td>216,161</td>
	</tr>
	<tr>
		<td>HyridSPAdes</td><td>SPAdes</td>
		<td>4,118,855*</td><td>4,839,764*</td><td>4,894,280*</td><td>30,045</td><td>40,806</td><td>20,453</td>
	</tr>
</table>

*`*` The `SPAdes` assembler includes a long tail of tiny fragments, which inflates the number of contigs severely.*

There are some pretty severe differences in the number of contigs obtained, although it's important to keep in mind that total number of contigs return is not *necessarily* a meaningful metric. How does this translate into bins per sample?

----

#### Bins and domain recovery

More interesting to us is how many bins (of modest quality) are recovered following binning and some basic filtering. Using the outputs of the `CheckM` and `GTDB-TK` runs in the previous section, I compiled a table summarising the findings.

For clarity, I have left zeros as empty columns. Where no bins were recovered for any sample, the row is struck out.

<table border='1'>
	<tr>
		<th></th><th></th><th></th><th colspan='3'>ONT mapping</th><th colspan='3'>Illumina mapping</th>
	</tr>
	<tr>
		<th>Data type</th><th>Assembler</th><th>Sample</th>
		<th>Total bins</th><th>Bacteria (>50%)</th><th>Archaea (>50%)</th>
		<th>Total bins</th><th>Bacteria (>50%)</th><th>Archaea (>50%)</th>
	</tr>
	<tr><th>Uncorrected</th><th>CANU</th><th>S4R3</th><th>3</th><th></th><th></th><th>9</th><th></th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>10</th><th>2</th><th></th><th>29</th><th>2</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th></th><th></th><th></th><th>8</th><th></th><th></th></tr>
	<tr><th></th><th>Flye</th><th>S4R3</th><th>4</th><th></th><th></th><th>11</th><th></th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>5</th><th>1</th><th></th><th>14</th><th>2</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>1</th><th></th><th></th><th>4</th><th></th><th></th></tr>
	<tr><th></th><th>MetaFlye</th><th>S4R3</th><th>4</th><th></th><th></th><th>10</th><th></th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>4</th><th>1</th><th></th><th>11</th><th>2</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>5</th><th></th><th></th><th>5</th><th></th><th></th></tr>
	<tr><th></th><th>MetaFlye (tuned)</th><th>S4R3</th><th>3</th><th></th><th></th><th>11</th><th></th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>5</th><th></th><th></th><th>14</th><th>3</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>5</th><th></th><th></th><th>4</th><th></th><th></th></tr>
	<tr><th></th><th>MiniASM</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th></tr>
	<tr><th></th><th>Shasta</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th></tr>
	<tr><th>FMLRC</th><th>CANU</th><th>S4R3</th><th>8</th><th>3</th><th></th><th>14</th><th>5</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>12</th><th>8</th><th></th><th>19</th><th>12</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>8</th><th>2</th><th></th><th>13</th><th>4</th><th></th></tr>
	<tr><th></th><th>Flye</th><th>S4R3</th><th>8</th><th>2</th><th></th><th>13</th><th>4</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>10</th><th>4</th><th></th><th>16</th><th>7</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>5</th><th>2</th><th></th><th>8</th><th>2</th><th></th></tr>
	<tr><th></th><th>MetaFlye</th><th>S4R3</th><th>4</th><th>1</th><th></th><th>12</th><th>4</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>9</th><th>4</th><th></th><th>16</th><th>6</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>3</th><th>2</th><th></th><th>9</th><th>4</th><th></th></tr>
	<tr><th></th><th>MetaFlye (tuned)</th><th>S4R3</th><th>5</th><th>2</th><th></th><th>12</th><th>5</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>11</th><th>2</th><th></th><th>16</th><th>7</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>4</th><th>2</th><th></th><th>8</th><th>4</th><th></th></tr>
	<tr><th></th><th>MiniASM</th><th>S4R3</th><th>4</th><th>2</th><th></th><th>4</th><th>1</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>5</th><th>4</th><th></th><th>7</th><th>3</th><th></th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>5</th><th>1</th><th></th><th>4</th><th>1</th><th></th></tr>
	<tr><th></th><th>Shasta</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th><th>-</th></tr>
	<tr><th>HyridSPAdes</th><th>SPAdes</th><th>S4R3</th><th>9</th><th>8</th><th></th><th>27</th><th>21</th><th></th></tr>
	<tr><th></th><th></th><th>S5R2</th><th>18</th><th>14</th><th></th><th>34</th><th>28</th><th>2</th></tr>
	<tr><th></th><th></th><th>S6R1</th><th>10</th><th>6</th><th></th><th>15</th><th>8</th><th></th></tr>
</table>

So it's pretty clear at this level of analysis that `SPAdes` yields the most, and highest quality MAGs. However, `SPAdes` cannot assembly long-read data with Illumina sequences for error correcting, so the comparison is really only valid against the error-corrected results for binning pipelines. In the absence of Illumina data, `CANU` appeared to recover the highest number of medium-quality (or greater) MAGs.

----

#### Testing the structure of the MAG genomes compared with a reference

TO DO...
