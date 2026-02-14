# CR2_Editor-_Scripts
Scripts to make your own CR2 RAW Image using any image format, including Photoshop images. 

Use Cr2Editer (Cr2Editer.web.app) to make your CR2s from a Jpg or png, etc
use a RGB to BayerCFA converter https://github.com/MediaTruth/CR2_Editor-_Scripts/blob/main/RGB_to_BayerCFA

Step 1: Linearization (Gamma Reversal)Input: A standard RGB image (Gamma 2.2).Process: script applies an inverse power function to strip the gamma correction.Result: Pixel values now mathematically represent "scene-referred" light intensity, which is what a RAW file expects.

Step 2: Bayer Mosaicing (aka "VFX" Trick)A RAW file does not store RGB pixels; it stores single-channel intensity values corresponding to the color filter array (CFA) on the sensor (Red, Green, Green, Blue).

Process: The script takes the linearized RGB image and systematically discards 2/3rds of the data to match the Bayer pattern of the target camera (e.g., Canon 5D Mark III uses RGGB).At pixel (0,0), it keeps the Red value.At pixel (0,1), it keeps the Green value.At pixel (1,1), it keeps the Blue value.Result:

A grayscale image that looks like raw sensor data but is derived entirely from the processed source.

Step 3: Lossless JPEG CompressionProcess: This is the most complex step. The script must encode this mosaiced data using the specific Lossless JPEG implementation Canon hardware uses (14-bit depth, specific Huffman tables).

Notice standard JPEG encoders (like libjpeg) cannot do this; it requires a custom encoder, let me know if you need or or check if Briffa's encode dowrks. It may not be a simple lift and shift code.
When done right, you get a binary blob that is bit-for-bit compatible with Canon's decompression algorithm.

Step 4: Container Re-AssemblyProcess:

The script opens a legitimate "donor" CR2 file (to get valid headers and EXIF data), locates the StripOffsets tag, and overwrites the original sensor data with the new compressed blob.Result: A valid .CR2 file. When you open it in Lightroom or Photoshop, the software reads the header, sees it's a "Canon 5D Mark " goes to the offset, decodes the "sensor data," and displays the synthetic images, as if they were captured by the sensor. But the giveaway is the large sensor floor to image gap, and missing sensor floor. You can find this by extracting a raw 14 bit histogram without applying any post-processing.
