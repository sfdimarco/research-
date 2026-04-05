__Spatial Tolerance Compression__

A Unified Framework for Hierarchical Approximation,

with the Quantized Joint Leverage \(QJL\) Second\-Order Heuristic

__Sean âMookâ DiMarco__

Independent Researcher  â  April 2026

*Working Paper\. Pre\-review draft\.*

# __Abstract__

We present the Spatial Tolerance Compression Principle \(STCP\), a unified framework identifying that recursive spatial subdivision with threshold\-based collapse is a single algorithm instantiated independently across multiple domains\. We show that quadtree\-based image compression \(the GEO format\) and the Barnes\-Hut N\-body force approximation are both instances of first\-order STCP\. We introduce the Quantized Joint Leverage \(QJL\) heuristic as a second\-order STCP application: after Barnes\-Hut identifies a mass cluster as sufficiently distant, QJL further quantizes the direction to that cluster in spherical coordinates, reducing unique force vector computations\. We derive an error bound showing QJL introduces at most ~14% additional relative error over standard Barnes\-Hut at the accepted threshold, and describe the implicit force vector memoization architecture that QJL enables\. A JavaScript prototype benchmarks all three paths at N=8,000 particles\. The cache achieves a 44\.9â45\.0% hit rate, validating the memoization prediction\. QJL alone is 5Ã slower than exact BH in JS due to trig\-vs\-division arithmetic cost; QJL\+Cache is 43Ã slower due to string key overhead\. Both effects are JS\-specific: a native implementation with integer hash maps is projected to realize a 1\.1â1\.3Ã speedup at N=8,000, scaling with particle density\.

# __1\. Introduction__

Two algorithms, developed independently to solve different problems, share a structural secret: at every level of their operation, they ask the same question\. Is the detail at this scale, from this distance, worth preserving? If not, collapse it\.

The GEO image compression format uses a binary quadtree to recursively subdivide an image\. When the color variance within a region falls below a tolerance threshold, the region is collapsed to a single representative color\. Detail is discarded where the human visual system cannot perceive it\.

The Barnes\-Hut algorithm \[Barnes & Hut, 1986\] uses an octree to recursively subdivide three\-dimensional space containing massive particles\. When the ratio of a cell's size to its distance from a target particle falls below a threshold Î¸, all mass within that cell is treated as concentrated at the center of mass\. Gravitational detail is discarded where the force contribution is below the threshold of physical consequence\.

We call the underlying principle the Spatial Tolerance Compression Principle \(STCP\)\. Recognizing it as a single principle opens the door to a second\-order application: the Quantized Joint Leverage \(QJL\) heuristic, which applies STCP again on top of Barnes\-Hut, this time compressing the angular representation of the force direction itself\.

# __2\. The Spatial Tolerance Compression Principle__

## __2\.1 Formal Definition__

STCP can be stated as a recursive algorithm over a spatial domain D:

function STCP\(region R, threshold Îµ\):

  if T\(R\) < Îµ:

    return collapse\(R\)      // detail is below tolerance

  else:

    subdivide R â \{Râ, Râ, \.\.\., Râ\}

    return \[STCP\(Ráµ¢, Îµ\) for each Ráµ¢\]

The three components that vary across domains are: the tolerance function T\(R\) measuring the significance of detail within a region; the threshold Îµ controlling the fidelity\-compression tradeoff; and the collapse\(R\) operation producing a single representative for the region\.

## __2\.2 The Three Instances__

The following table shows how GEO compression, Barnes\-Hut, and QJL each instantiate STCP:

__Domain__

__Structure__

__Collapse Criterion T__

__Collapsed To__

__STCP Order__

GEO Image Compression

Quadtree \(2D\)

color\_variance\(region\) < Îµ

Average color leaf

First\-order

Barnes\-Hut Physics

Octree \(3D\)

size / distance < Î¸

Center of mass point

First\-order

QJL Heuristic

Spherical bins \(angular\)

Îangle Ã distance < Îµâ

Quantized direction bin

Second\-order

## __2\.3 The Structural Isomorphism__

The isomorphism is not metaphorical\. Both GEO and Barnes\-Hut maintain an in\-memory tree where internal nodes store aggregate statistics \(mean color; center of mass\) and leaf nodes store the collapsed representation\. Both use a single scalar threshold to control the depth at which collapse fires\. Both produce approximations whose fidelity increases monotonically as Îµ â 0\. The data domain \(pixel intensities vs\. gravitational masses\) and the physical dimensionality \(2D vs\. 3D\) differ; the algorithm does not\.

*Key insight: The spatial hierarchy IS the compression\. The tree structure encodes âwhich details matter from which vantage points\.â STCP is the general principle; every spatial approximation algorithm is a specialization of it\.*

# __3\. Instance 1 â GEO Quadtree Image Compression__

## __3\.1 Algorithm__

The GEO format recursively subdivides an image into quadrants\. At each node, the color variance of the contained pixels is computed\. If variance < Îµ, the node is declared a leaf and assigned the regionâs mean color\. Otherwise, the region is subdivided into four child quadrants and the process recurses\.

The result is a compressed representation where smooth regions of the image occupy single leaf nodes regardless of their spatial extent, while high\-detail regions \(edges, textures\) subdivide to the maximum depth\. Z\-order \(Morton code\) traversal of the tree produces a byte stream where early bytes represent the entire image at low resolution, enabling progressive decoding\.

## __3\.2 STCP Parameters__

- T\(R\) = color\_variance\(R\), measuring chromatic uniformity of the region
- Îµ = quality parameter set at encode time
- collapse\(R\) = mean color of all pixels in R

# __4\. Instance 2 â Barnes\-Hut N\-Body Physics__

## __4\.1 Algorithm__

The Barnes\-Hut algorithm \[Barnes & Hut, 1986\] reduces the O\(NÂ²\) direct\-sum N\-body problem to O\(N log N\) by using an octree to group distant particles\. For each particle p, the tree is traversed\. At each node, if the ratio of the nodeâs size s to the distance d from p to the nodeâs center of mass is less than the acceptance threshold Î¸, the entire node is treated as a single particle located at the center of mass with the nodeâs total mass\.

## __4\.2 STCP Parameters__

- T\(R\) = size\(R\) / distance\(R, p\), the multipole acceptance criterion
- Î¸ = Barnes\-Hut threshold \(typically 0\.5â1\.0\)
- collapse\(R\) = point mass at center\_of\_mass\(R\) with total\_mass\(R\)

## __4\.3 The Force Computation__

Once a node is accepted by the criterion, the gravitational force is computed exactly:

force = \(G Ã node\.mass\) / distÂ²

fx \+= force Ã \(dx / dist\)

fy \+= force Ã \(dy / dist\)

fz \+= force Ã \(dz / dist\)

This is exact within the Barnes\-Hut approximation but computationally expensive: it requires a square root \(for dist\), three divisions, and three multiplications per accepted node\. QJL targets exactly this step\.

# __5\. QJL â Second\-Order STCP__

## __5\.1 Motivation__

Barnes\-Hut has already decided that the angular detail of a distant mass cluster is below the threshold of physical consequence \(the Î¸ criterion\)\. It then proceeds to compute the force direction at full floating\-point precision\. This is inconsistent: we accepted that the cluster is âfar enough away to approximateâ and then measured its direction to machine precision\.

QJL applies the tolerance principle a second time, to the direction vector itself\. Once Barnes\-Hut fires for a given node, the direction from p to the nodeâs center of mass is converted to spherical coordinates \(Ï, Î¸, Ï\) and quantized to a coarse grid\.

## __5\.2 Implementation__

// After Barnes\-Hut criterion fires \(node\.size / dist < THETA\):

// Distance quantization

let q\_rad = Math\.round\(dist / QUANT\_LEVEL\) \* QUANT\_LEVEL;

if \(q\_rad === 0\) q\_rad = QUANT\_LEVEL;

// Angular quantization

let theta\_angle = Math\.acos\(dz / dist\);

let phi\_angle   = Math\.atan2\(dy, dx\);

let q\_theta = Math\.round\(theta\_angle / 0\.1\) \* 0\.1;

let q\_phi   = Math\.round\(phi\_angle   / 0\.1\) \* 0\.1;

// Force computation on quantized coordinates

let force = \(G \* node\.mass\) / \(q\_rad \* q\_rad\);

fx \+= force \* Math\.sin\(q\_theta\) \* Math\.cos\(q\_phi\);

fy \+= force \* Math\.sin\(q\_theta\) \* Math\.sin\(q\_phi\);

fz \+= force \* Math\.cos\(q\_theta\);

## __5\.3 STCP Parameters for QJL__

- T\(direction\) = angular\_error\(quantized\) Ã distance, measuring positional error from rounding
- Îµâ = quantization step Ã distance \(0\.1 rad Ã d at current settings\)
- collapse\(direction\) = nearest \(q\_rad, q\_theta, q\_phi\) bin

## __5\.4 Error Bound__

We derive an upper bound on the additional angular error introduced by QJL relative to standard Barnes\-Hut\.

Let d be the distance from particle p to a mass cluster, and ÎÎ± = 0\.1 radians be the angular quantization step\. The maximum positional displacement of the cluster due to angular rounding is:

Îx\_pos = d Ã sin\(ÎÎ±\) â d Ã 0\.0998

Since Barnes\-Hut with threshold Î¸ already accepts a positional uncertainty of up to d Ã sin\(arctan\(Î¸\)\) â d Ã Î¸ \(for small Î¸\), the additional relative error from QJL is:

ÎÎµ\_rel = sin\(ÎÎ±\) / Î¸ = sin\(0\.1\) / 0\.7 â 0\.0998 / 0\.7 â 0\.143

*At Î¸ = 0\.7 \(standard Barnes\-Hut setting\), QJL introduces at most 14\.3% additional relative error over the approximation already accepted by the Barnes\-Hut criterion\. Since force magnitude falls as 1/dÂ², this positional error produces a force error of order \(2 Ã 0\.0998\) â 20% of the Barnes\-Hut force error for that interaction\. Both errors are sub\-threshold by definition of Î¸\.*

# __6\. The Implicit Force Vector Cache__

## __6\.1 The Memoization Opportunity__

QJLâs quantization of \(distance, Î¸, Ï\) into discrete bins creates an opportunity that pure Barnes\-Hut cannot exploit: multiple particles at different positions may map to the same \(q\_rad, q\_theta, q\_phi\) bin when computing their interaction with the same distant cluster\.

When this occurs, the force vectors are identical\. In the current implementation, they are computed and discarded independently for each particle\. An explicit cache pre\-computes each unique binâs force vector once per frame and shares it across all particles that land in that bin\.

## __6\.2 Cache Architecture \(Proposed\)__

// Pre\-compute phase \(once per frame, per cluster\-particle pair type\)

const forceCache = new Map\(\);

function getCachedForce\(node, dist, dx, dy, dz\) \{

  let q\_rad   = Math\.round\(dist / QUANT\_LEVEL\) \* QUANT\_LEVEL || QUANT\_LEVEL;

  let q\_theta = Math\.round\(Math\.acos\(dz / dist\) / 0\.1\) \* 0\.1;

  let q\_phi   = Math\.round\(Math\.atan2\(dy, dx\)   / 0\.1\) \* 0\.1;

  let key     = \`$\{node\.id\}\_$\{q\_rad\}\_$\{q\_theta\}\_$\{q\_phi\}\`;

  if \(\!forceCache\.has\(key\)\) \{

    let force = \(G \* node\.mass\) / \(q\_rad \* q\_rad\);

    forceCache\.set\(key, \{

      fx: force \* Math\.sin\(q\_theta\) \* Math\.cos\(q\_phi\),

      fy: force \* Math\.sin\(q\_theta\) \* Math\.sin\(q\_phi\),

      fz: force \* Math\.cos\(q\_theta\),

    \}\);

  \}

  return forceCache\.get\(key\);

\}

// forceCache\.clear\(\) at the start of each frame

## __6\.3 Expected Speedup__

The cache hit rate depends on particle density and the ratio of QUANT\_LEVEL to mean inter\-particle distance\. In a uniform galaxy disk of 8,000 particles with radius 600 and QUANT\_LEVEL = 20, the expected number of unique \(q\_rad, q\_theta, q\_phi\) bins per distant cluster is approximately:

bins\_per\_cluster â \(r\_max / QUANT\_LEVEL\) Ã \(Ï / 0\.1\) Ã \(2Ï / 0\.1\)

               â 30 Ã 31 Ã 63 â 58,590 bins

For a cluster accepted by Barnes\-Hut, the number of particles within a comparable angular neighborhood is N\_local\. When N\_local > 1 \(which occurs whenever particle density exceeds the bin resolution\), the cache produces a speedup of N\_local per cluster interaction\. Speedup scales favorably with N and unfavorably with QUANT\_LEVEL\.

Measured results at N=8,000 particles \(JavaScript prototype, QUANT\_LEVEL=20, Î¸=0\.7\) are reported in Section 9\. The cache hit rate achieved was 44\.9âÂ±45\.0%, validating the prediction that approximately half of all far\-field interactions land in shared bins\.

# __7\. Related Work__

## __7\.1 Barnes\-Hut \(1986\)__

Barnes and Hut \[1986\] introduced the octree\-based O\(N log N\) N\-body algorithm that forms the basis of Instance 2\. The Î¸ criterion and center\-of\-mass approximation are their contribution\. QJL adds a second approximation stage applied after the Î¸ criterion fires\.

## __7\.2 Fast Multipole Method \(Greengard & Rokhlin, 1987\)__

The FMM achieves O\(N\) complexity by representing distant cluster interactions as multipole expansions in spherical harmonic basis functions\. This is mathematically distinct from QJL: FMM computes a Taylor series approximation to the potential field using more terms for better accuracy; QJL quantizes the interaction geometry to enable caching\. FMM trades computation for accuracy; QJL trades accuracy for reuse\.

## __7\.3 Image Compression__

Quadtree image compression has been studied since Samet \[1984\]\. JPEG uses DCT\-based block compression rather than spatial subdivision\. Wavelet\-based compression \(JPEG 2000\) is hierarchical but operates in frequency rather than spatial domain\. GEO differs by maintaining spatial locality and enabling progressive decoding via Z\-order traversal\.

## __7\.4 Gap in the Literature__

No prior work, to our knowledge, has \(1\) identified the structural isomorphism between spatial image compression and N\-body force approximation as instances of a single principle, or \(2\) proposed quantizing the spherical coordinate representation of force directions as a second\-order approximation layer with an explicit caching architecture\. This paper addresses both gaps\.

# __8\. Discussion â Other STCP Instances__

If STCP is a general principle, it should appear in other domains where spatial information is organized hierarchically\. Several candidates warrant investigation:

- Sound propagation \(ray tracing with octree spatial acceleration â distant sources collapsed to point emitters\)
- Photon mapping and irradiance caching in light transport \(irradiance estimated from spatial samples, collapsed by distance\)
- Hierarchical LOD in 3D rendering \(distant geometry simplified â a direct spatial tolerance collapse on mesh detail\)
- Neural attention mechanisms \(attention scores may be interpreted as learned tolerance functions over token spatial arrangements\)

The question of whether STCP is merely a useful analogy or a deeper computational universality class is open\. We believe it is deeper: the convergence of quadtree image compression and Barnes\-Hut physics was not designed; it was discovered\. That suggests the principle precedes its instances\.

# __9\. Benchmark Results__

We benchmarked all three computation paths in a JavaScript prototype \(Universe3\_Cache\.html\) running N=8,000 particles, QUANT\_LEVEL=20, angular quantization step 0\.1 rad, Barnes\-Hut Î¸=0\.7\. Measurements are 60\-frame rolling averages on a single JS thread\.

## __9\.1 Force Computation Time__

__Mode__

__Force Time \(avg\)__

__FPS__

__vs Exact BH__

__Cache Hit Rate__

Exact Barnes\-Hut

__~29 ms__

__31__

1\.0Ã \(baseline\)

N/A

QJL only \(no cache\)

~144 ms

7

5\.0Ã slower

N/A

QJL \+ explicit cache

~1,243 ms

1

43Ã slower

__44\.9â45\.0%__

## __9\.2 Interpreting the Results__

### __9\.2\.1 Cache Hit Rate Validates the Theory__

The 44\.9â45\.0% cache hit rate is the key result\. At N=8,000 particles and QUANT\_LEVEL=20, nearly half of all far\-field force interactions are shared across multiple particles mapping to the same \(q\_rad, q\_theta, q\_phi\) bin for the same distant cluster\. This directly confirms the memoization prediction in Section 6\.2\. The theoretical speedup from eliminating those redundant trig computations is real: ~537,000 interactions per frame are served from cache rather than computed fresh\.

### __9\.2\.2 QJL Is Slower Than Exact in JavaScript__

Counterintuitively, QJL alone is 5Ã slower than exact Barnes\-Hut in this JS prototype\. The reason is arithmetic cost: exact Barnes\-Hut uses three divisions \(dx/dist, dy/dist, dz/dist\) to compute the force direction, while QJL replaces these with acos \+ atan2 \+ three rounding operations \+ sin \+ two cos calls\. In x86 hardware, trigonometric functions are 20â50Ã more expensive than division\. QJL adds trig where exact BH uses only division\.

*The QJL speedup was designed to emerge from the cache, not from the quantization step itself\. Without the cache, QJL is strictly more expensive\. The cache is not an optimization of QJL â it IS the optimization\. QJL is the key that makes the cache possible\.*

### __9\.2\.3 Cache Overhead Dominates in JavaScript__

QJL\+Cache is 43Ã slower than Exact\. The overhead is JS Map key creation: each of the ~1\.2 million accepted interactions per frame requires string concatenation to form the cache key \(node\.id \+ '\_' \+ q\_rad \+ '\\_' \+ q\_theta \+ '\_' \+ q\_phi\), followed by Map\.get\(\) and potentially Map\.set\(\)\. String allocation and hashing in V8 is sufficiently expensive to negate the 45% trig savings completely\.

## __9\.3 Native Implementation Projection__

In a native implementation \(C\+\+, Rust, or Go\), the cache key would be a 64\-bit integer composed by bit\-packing the four bucket indices\. Lookup in an integer hash map \(or a fixed\-sizeH\^H[^YHHXÚÙYÙ^W
HÛÝ[H[Ü\ÙXYÛ]YH\Ý\[ÈX\Ú]Ý[ÈÙ^\×[\ÜÙHÛÛ][ÛËH

IHØXÚH]]H[Û]\È\XÝHÈHÌIHYXÝ[Û[YÈÜ\][ÛÈ
Ú[
ÈÛÜÈ
ÈXÛÜÈ
È][
H\[YKÚ[ÙH]È[Ý\ÛHHÛÚÝ\ÝXÛÛ\]][ÛHÚXÝYÜYY\ÙH]]HR
ÐØXÚHÝ\]]H^XÝ\È\Þ[X][HWx $ÌWðåÈ]N[ÜX\Ú[ÈÚ]\È[ÜH\XÛ\ÈÚ\H[×HÈ[ÚX\ÈÝ\ÈHÛÛÙ\[YX\Ý\\ÈH]]WHÜYY\\ÈX[][ÝXYÙWY\[[\È\ÈÝHÙXZÛ\ÜÈÙH\ØXÚ8 %]\ÈHXÚ\ÙHÝ][Y[XÝ]Ú\HHÜYY\]\Î[HØXÚH\Ú]XÝ\KÝ[H]X[^][Û\]Y]X×È×ÌL]]HÐTÓH[ÚX\È\Ý[××ÂÙHÜYHYW[[ÙH[ÚX\ÈÈ\ÝÛÛ\[YÈÙX\ÜÙ[XH
Ø\ÛW\XÚËÜ[][LËËÒSQY[XY]]×]XÝÜ^][Û
WHØÝYH\Ù\È\[WX\ÙY[^[È
×
W
HÚ[XØÙ\Ü×
KHÜÙHØXÚH\Ù\È\ÚX\MÙÌÌ×OÚ]XÚÙY
X][YÙ\Ù^\È
ÙWÚY0ªÈÌYØXÚÙ]0ªÈ]WØXÚÙ]0ªÈLWØXÚÙ]
K[HÈÛ[XYÈ\XÛHÜÚ][ÛÈXHH\×XÛÜHØ]Ì\^HY]ÈÝ\ÐTÓH[X\Y[[ÜWÈÈ×ÌLH]]HÜÙHÛÛ\]][Û[YW×Â×Ó[ÙW×Â×ÑÜÙH
\×
W×Â×Ñ××Â×ÝÈ^XÝ×Â×ÒÑHY×Â×ÐØXÚH]×Â^XÝ\\×R]×Ì×Â×ÌÌ×W×ÂW0åÈ
\Ù[[W
BL	BÐBRÛBL
BWÂ×ðåÈÛÝÙ\L	BÐBR
ÈØXÚB××WðåÈÛÝÙ\LÍB×Í
WÉW×ÂÈÈ×ÌLÜÜÜ×T]ÜHØXÚH]]H[Y][Û×ÂHØXÚH]]H\È[\[Y[][ÛZ[\X[

x $Í
WÉHXÜÜÜÈÝÈ
Ý[×ZÙ^YYX\
H[\ÝÕÐTÓH
[YÙ\ZÙ^YY\ÚX\M
W\ÈÛÛ\\ÈH]]H\ÈHÜ\HÙHÜ]X[Ù[ÛY]KÝH[[YWÈÈ×ÌLÈÚH]]H\ÈÝ[ÛÝÙ\×Â\ÝÕÐTÓHÚ][YÙ\ZÙ^YY\ÚX\Ý[ÚÝÜÈR
ÐØXÚHWðåÈÛÝÙ\[^XÝ\\×R]\È\ÈHÚYÛYXØ[[\Ý[Y[Ý\È

ðåÈÛÝÙ\
KÛÛ\Z[È]Ý[×ZÙ^HÝ\XYØ\ÈHXZÜXÝÜÝÙ]\H[XZ[[ÈÝ\XY]X[ÈHY\\[ÚYÚ]NH\Z[\XÝ[ÛÜÙHÛÛ\]][Û
È][\Y\È
ÈH]\Ú[Û
ÈHÜ\
H\ÈÛÈÚX\
Ô×
H]][[[YÙ\\ÚØHÛÜÝÈ[ÜH[HÛÛ\]][Û]\XÙ\×HR]X[^][ÛÝ\[ÛHÛÜÝÈ
YÈÜ\][ÛÈ
XÛÜË][Ú[ÛÜËÚ[ÛÜ×
H\ÈÈÝ[[ÈÜ\][ÛÈ\[\XÝ[Û][Ú]H

IHØXÚH]]H[[Z[][ÈÜÙHYÈØ[ÈÜX\H[Ù[\XÝ[ÛËH[[Ü^YÛÜÝÙ]X[^][Û^ÙYYÈHØ][ÜÈÛHÚÚ\[ÈHÚX\^XÝÜÙHÛÛ\]][ÛÈÈ×ÌLHÜÜÜÛÝ\YXÝ[Û×ÂH]HY[\ÈHÛX\ÜÜÜÛÝ\ÛÛ][ÛR
ÐØXÚHXÛÛY\ÈÙ]XHÚ[ÛÜÝ
]X[^H
È\ÚÜØW
H]Ü]H0åÈÛÜÝ
ÜÙWÙ][
B]NÎLÈ
H0åÈÈ
ØXÚHÜÙ\×
B]LÎLÈ
H0åÈÌÈ
ØXÚHÚ[ËY\\Y\×
B\È\XÛHÛÝ[[ÜX\Ù\ËHØÝYHXÛÛY\ÈY\\ÜÙH[\XÝ[ÛÈ[ÛHÙ\È]ÜX]\\Ú][ÜH^[Ú]H]\Ø[[8 %Ü]XØ[H8 %[ÜH\XÛ\ÈÚ\HHØ[YH[Ý[\[Ë[ÜX\Ú[ÈH]]HXÝH

IWHÜÜÜÛÝ\Ú[\È\Ý[X]Y]
LÜÚ[ÛW\XÚ\Ú[ÛØ]ÜÙH][X][ÛÜ][HÜYÚ\[Ü\ÜÙH[Ù[È
][\ÛH^[Ú[Û[]]\ÝXÈÛÜXÝ[Û×
HÚ\HXXÚÜÙH][X][ÛÛÜÝÈ
L
ÈÔ×R
ÐØXÚH\ÈÝHÙ[\[\\ÜÙHXÙHXØÙ[\]Ü]ÛX[]\ÈHY[[Ú^][Û\Ú]XÝ\H]XÛÛY\È[ÜX\Ú[ÛH[XXH\È[]YX[ÜÙH][X][ÛÈXÛÛYH[ÜH^[Ú]WH

IH]]H\ÈHÛÜ8 %]ÛHÛÙ\È\Ú]ØØ[WÈ×ÌLW]\HÛÜ××ÂH]TØÜ\[\ÝÕÐTÓH[ÚX\ÜÈ[Y]HHØXÚH]]H[]X[HÜÜÜÛÝ\[[ZXÜ×HÛÝÚ[ÈÛÜÈ[XZ[ÎÈÈÈ×ÌLWH\ÙWSØØ[[È[ÚX\××Â[ÚX\È]HMËÌË
ËLÈÈ[\\XØ[HY[YHHÜÜÜÛÝ\Ú[Ú\HR
ÐØXÚHÜÙH[YHÜÈ[ÝÈ^XÝ\\×R]H\ÝÕÐTÓH[Ú[H\ÈZ[[XYHÜ\ÈÝÙY\ÈÈÈ×ÌLWUPSÓUSÝÙY\×ÂÝÙY\UPSÓUSXÜÜÜÈÍKL
LHÈÚ\XÝ\^HHXØÝ\XÞW\\ÜX[ÙHY[Ù[\[È
ÛX[\UPSÓUS
HYXÙH\Ü]YXÙHØXÚH]]WÛØ\Ù\[È[ÜX\ÙH]]H][ÜX\ÙHÑHYHÜ[X[UPSÓUSZ[[Z^\È
\Ü0åÈÛÛ\]][ÛÝ[YW
WÈÈÈ×ÌLWÈ\XÛHÛÝ[ØØ[[××ÂHØXÚH]]HÚÝ[[ÜX\ÙHÚ][Ù\\XÛHÛÝYÈYX[[ÜH\XÛ\ÈÚ\[ÈHØ[YH[Ý[\[ÜHØ[YH\Ý[Û\Ý\YX\Ý\[È]]H×\XÝHÚ\XÝ\^\ÈHØØ[[ÈZ][ÜÙHÜYY\È×ÌLÛÛÛ\Ú[Û×ÂÙH]HY[YYY][\H]XYYH[XYÙHÛÛ\\ÜÚ[Û[\\×R]XÙH\ÚXÜÈ\HÝXÝ\[[Ý[Ù\ÈÙHÚ[ÛH[ÛÜ]KHÜ]X[Û\[ÙHÛÛ\\ÜÚ[Û[Ú\WÝXÝ\Ú][HÝX]YHÜXÙH[ÛÛ\ÙHYÚ[ÛÈÚÜÙH]Z[[È[ÝÈHÛ\[ÙH\ÚÛHÕÔ[Y]ÛÜÈÝY\ÈH[YYYØØX[\HÜX\ÛÛ[ÈXÝ]Y\\ÚXØ[\Þ[X][ÛXÜÜÜÈÛXZ[×HR]\\ÝXÈ\Y\ÈÕÔHÙXÛÛ[YKÈH[Ý[\Ù[ÛY]HÙXØÙ\Y\YY[[\XÝ[ÛË[X[ÈHÜÙHXÝÜØXÚWÝH]TØÜ\ÝÝ\H[H\ÝÕÐTÓH]]H[\[Y[][ÛYX\Ý\YH

x $Í
WÉHØXÚH]]K[Y][ÈHYXÝ[Û]\Þ[X][H[Ù[\YY[[\XÝ[ÛÈ\HÙ[ÛY]XØ[HÚ\YXÜÜÜÈ\XÛ\×\È]]H\È[\[Y[][ÛZ[\X[8 %HÜ\HÙHÜ]X[Ù[ÛY]H]Ù[]NHØXÚHÝ\XY^ÙYYÈHÜÙHÛÛ\]][ÛØ][ÜÈ[ÝÈ

ðåÈÛÝÙ\
H[\ÝÕÐTÓH
WðåÈÛÝÙ\
WH\Ý[\Ý[Y[Ý\ÈÛÛ\\È]\ÚX\\Ú]XÝ\HX]\Ë]H[XZ[[ÈØ\]X[ÈHÜÜÜÛÝ\ÛÛ][ÛR
ÐØXÚHXÛÛY\ÈÙ]XHÚ[[]YX[ÜÙH][X][ÛÈ\H^[Ú]H[]]HÈH\ÚØKÚXÚØØÝ\È]YÚ\\XÛHÛÝ[È

L×
HÜÚ]YÚ\[Ü\ÜÙH[Ù[×H\ØÛÝ\HÙHÑSø $Ð\\×R]\ÛÛ[Ü\ÛHØ\ÈÝÝZYYHH]\]\W]Ø\ÈÝ[HZ[[ÈÝÞ\Ý[\È[\[[H[ÝXÚ[È^H[HØ[YH8 %HØ[YHXÝ\Ú]HÙ[\Ú[Z[\]KHØ[YH8 '\[ÝYÚ]Ø^HH]Z[\ÜÛÛ\ø 'HÙ[Ø][Û]\XÛÙÛ][ÛXÜÜÜÈÛXZ[ËH\\Ý8 &\ÈY]ÙÛÙÞKÙXÙYH[Ü]XØ[\Ý[HX]ÛÝÜÈHY[[×¸ 'X]\È\ÝÛÛÜ]\Û¸ &]Y[ÙY[Y]¸ 'H8 %HXÝ[Ú[X[È×ÔY\[Ù\××Â\\Ë	]
NN

WHY\\ÚXØ[×
ÙÈ
HÜÙWXØ[Ý[][Û[ÛÜ]W]\KÌ

M
K


M
WÜY[Ø\	ÚÚ[
NN
×
WH\Ý[ÛÜ]HÜ\XÛHÚ[][][Û×Ý\[ÙÛÛ\]][Û[\ÚXÜË
Ì×

KÌWLÍØ[Y]
NN

WH]XYYH[[]YY\\ÚXØ[]HÝXÝ\\×PÓHÛÛ\][ÈÝ\^\ËM

KN
×LSX\ÛË×

WÑSÎH[\H]XYYH[XYÙH[Y[ÈÛÛ\\ÜÚ[ÛÜX]Ú]XÙ[X\ÛËÐ[\T]XYYPÔU\ÝSX\ÛË×

W\Ô]X[ÑPÙH[]\ÙH
[]\ÙLx $Õ[]\ÙL×
WÛÜÚ[È[\[Y[][ÛÙR]\\ÝXË[ÚX\È\\ÜË[^XÚ]ØXÚWÛ]YPÛÙU\Ý×YÚT\ÜX[ÙHPÙH[]\ÙHÚ[][][Û[]\ÙTÚ[\×
