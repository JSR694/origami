# origami

Building and using OCR tools can be hard. origami is a low-overhead suite of batch tools for OCR processing to make one’s life easier. It consists of various minimalistic batches and tools useful in building training data for training models and/or generating OCR output from existing models.

# Installation 

```
conda create --name origami python=3.7 -c defaults -c conda-forge --file origami/requirements/conda.txt
conda activate origami
pip install -r origami/requirements/pip.txt
```

## General Usage

```
cd /path/to/origami
python -m origami.batch.detect.segment
```

All command line tools will give you help information on their arguments when called as above.

The given data path should contain processed pages as images. Generated data is put into the same path.  Images may be structured into any hierarchy of sub folders.

Batch processes can be run concurrently.

For generating ground truth for training an OCR engine from a corpus, we suggest this general process:

* Run batches up to `lines` on your page images.
* Sample random lines using `origami.tool.sample`.
* Fine tune your training corpus using `origami.tool.pick`.
* Annotate using `origami.tool.annotate`.
* Export annotations using `origami.tool.export`.
* Train your OCR model.

## Batches

Given an OCR model, the necessary order of batches for performing OCR for a folder of documents is:

<table>
<tr>
<td>1</td>
<td>segment</td>
</tr>
<tr>
<td>2</td>
<td>contours</td>
</tr>
<tr>
<td>3</td>
<td>warp</td>
</tr>
<tr>
<td>4</td>
<td>dewarp</td>
</tr>
<tr>
<td>5</td>
<td>layout</td>
</tr>
<tr>
<td>6</td>
<td>lines</td>
</tr>
<tr>
<td>7</td>
<td>order</td>
</tr>
<tr>
<td>8</td>
<td>ocr</td>
</tr>
<tr>
<td>9</td>
<td>compose</td>
</tr>
</table>

<dl>
  <dt>origami.batch.detect.segment</dt>
  <dd>needs: images</dd>
  <dd>⁂ Perform segmentation (e.g. separation into text and background) on all images using a neural network model. By default, this uses <a href="https://github.com/poke1024/bbz-segment">origami’s own model.</a>. The predicted classes and labels are embedded in the downloaded model.</dd>
</dl>

<dl>
  <dt>origami.batch.detect.contours</dt>
  <dd>needs: images, binarize, segment</dd>
  <dd>⁂ From the segmentation, detects connected components to produce vectorized polygonal contours for blocks and separator lines. 
  Uses a couple of rule-based approaches to fix some issues inherent in pixel-based segmentation (see --region-spread,
  --ink-spread and --ink-opening for details). Note that the separator detection is very slow and still work in progress.</dd>
</dl>

<dl>
  <dt>origami.batch.detect.lines</dt>
  <dd>needs: images, segment</dd>
  <dd>⁂ Detects baselines and line boundaries for each text line. For details, see  command line arguments. </dd>
</dl>

<dl>
  <dt>origami.batch.detect.ocr</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Performs OCR on each detected line using the specified Calamari OCR model. Note that the binarization
  you can specify here in independent of the one performed in origami.batch.detect.binarize.</dd>
</dl>

<dl>
  <dt>origami.batch.detect.xycut</dt>
  <dd>needs: images, contours</dd>
  <dd>⁂ Tries to find a reading order using a variant of the XY Cut algorithm.</dd>
</dl>

<dl>
  <dt>origami.batch.detect.compose</dt>
  <dd>needs: images, lines, ocr, xycut</dd>
  <dd>⁂ Composes text into one file using the detected reading order.</dd>
</dl>

<dl>
  <dt>origami.batch.detect.stats</dt>
  <dd>needs: nothing</dd>
  <dd>⁂ Prints out statistics on computed artifacts and errors. This is useful for
  understanding how many pages for processed, and for which stages this processing
  is finished.</dd>
</dl>

## Debugging

<dl>
  <dt>origami.batch.annotate.contours</dt>
  <dd>needs: stages 1, 2 (and maybe more, depending on `--stage`)</dd>
  <dd>⁂ Produces debug images for understanding the result of the contours batch stage.
  <img src="/docs/img/sample-2436020X_1925-02-27_70_98_009.debug.contours.jpg"></dd>
</dl>

<dl>
  <dt>origami.batch.annotate.lines</dt>
  <dd>needs: stages 1 - 6</dd>
  <dd>⁂ Produces debug images for understanding the line detection stage.
  <img src="/docs/img/sample-SNP2436020X-18720410-1-12-0-0.lines.jpg">
  </dd>
</dl>

<dl>
  <dt>origami.batch.annotate.layout</dt>
  <dd>needs: stages 1 - 7</dd>
  <dd>⁂ Produces debug images for understanding the result of the layout and order
  batch stage.</dd>
</dl>

## Tools

<dl>
  <dt>origami.tool.annotate</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Tool for annotating, viewing and searching for ground truth. <img src="/docs/img/sample-annotation.jpg"></dd>
</dl>

<dl>
  <dt>origami.tool.pick</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Tool for adding or removing single lines from the ground truth for fine tuning. <img src="/docs/img/sample-linepick.jpg"></dd>
</dl>

<dl>
  <dt>origami.tool.sample</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Create a new annotation database by randomly sampling lines from a corpus. The details of sampling (numbers of items
  for each segmentation label type per page) can be specified. Allows import of transcriptions stored in accompanying PageXML.
  See command line help for more details.</dd>
</dl>

<dl>
  <dt>origami.tool.schema</dt>
  <dd>⁂ Run an annotation normalization schema on the given ground truth text files.</dd>
</dl>

<dl>
  <dt>origami.tool.export</dt>
  <dd>⁂ From the given annotation database, export line images of the specified height and binarization together with accompanying
    ground truth text files. Annotation normalization through a schema is supported. Use this command to generate training data for
    <a href="https://github.com/Calamari-OCR/calamari">Calamari</a>. See command line for details.</dd>
</dl>

<dl>
  <dt>origami.tool.xycut</dt>
  <dd>⁂ Debug internal X-Y cut implementation.</dd>
</dl>

<dl>
  <dt>origami.batch.export.lines (debugging only)</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Export images of lines detected during lines batch.</dd>
</dl>

<dl>
  <dt>origami.batch.export.pagexml  (debugging only)</dt>
  <dd>needs: images, lines</dd>
  <dd>⁂ Export polygons of lines detected during lines batch as PageXML.</dd>
</dl>


## How to create ground truth

You first need to detect lines in your pages: in order to do this, run `origami.batch.detect.binarize`,
`origami.batch.detect.segment`, `origami.batch.detect.contours` and `origami.batch.detect.lines` on your
`DATA_PATH` that contains your page images.

Now run `python -m origami.tool.sample DATA_PATH` (look at the `-s` parameter to specify which content
you want to annotate) to generate an `annotations.db` with a sample of lines. You can now run
`python -m origami.tool.pick DATA_PATH` to extend this sample with hand-picked lines from various pages
(optional). Finally, run `python -m origami.tool.annotate DATA_PATH`, to edit and review transcriptions.
