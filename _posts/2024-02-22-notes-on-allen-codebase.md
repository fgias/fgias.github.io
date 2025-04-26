---
title: "Notes for New Developers on LHCb’s Allen Codebase"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - CERN
  - LHCb
  - GPU
comments: true
---

This post is meant to give extra comments and notes on the Allen codebase. For more introductory materials, see the Allen [GitLab](https://gitlab.cern.ch/lhcb/Allen/) and [documentation](https://allen-doc.docs.cern.ch/). The CUDA GNU debugger is highly recommended for exploring and understanding the codebase.

## Structure of Data in Allen

Essentially, data in Allen are stored in 1-dimensional arrays. Therefore, if we have many events, we need "offsets" to know where specifically the data of interest can be found within this 1-dimensional array.

## Pointers to Data in Allen

For example, let's look at [`SearchByTriplet.cu`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/search_by_triplet/src/SearchByTriplet.cu):

```cpp
__global__ void velo_search_by_triplet::velo_search_by_triplet(
  velo_search_by_triplet::Parameters parameters,
  const VeloGeometry* dev_velo_geometry)
{
  // Shared memory size is a constant, enough to fit information about three module pairs.
  __shared__ Velo::ModulePair module_pair_data[3];

  // Initialize event number and number of events based on kernel invoking parameters
  const unsigned event_number = parameters.dev_event_list[blockIdx.x];
  const unsigned number_of_events = parameters.dev_number_of_events[0];

  // Pointers to data within the event
  const unsigned total_estimated_number_of_clusters =
    parameters.dev_offsets_estimated_input_size[Velo::Constants::n_module_pairs * number_of_events];
  const unsigned* module_hit_start =
    parameters.dev_offsets_estimated_input_size + event_number * Velo::Constants::n_module_pairs;
  const unsigned* module_hit_num = parameters.dev_module_cluster_num + event_number * Velo::Constants::n_module_pairs;
  const unsigned hit_offset = module_hit_start[0];

  const auto velo_cluster_container =
    Velo::ConstClusters {parameters.dev_sorted_velo_cluster_container, total_estimated_number_of_clusters, hit_offset};

  const unsigned tracks_offset = Velo::track_offset(parameters.dev_offsets_estimated_input_size, event_number);
  Velo::TrackHits* tracks = parameters.dev_tracks + tracks_offset;
  Velo::TrackletHits* three_hit_tracks = parameters.dev_three_hit_tracks + tracks_offset;

  Velo::TrackletHits* tracklets = parameters.dev_tracklets + event_number * Velo::Constants::max_tracks_to_follow;
  unsigned* tracks_to_follow = parameters.dev_tracks_to_follow + event_number * Velo::Constants::max_tracks_to_follow;

  bool* hit_used = parameters.dev_hit_used + hit_offset;
  uint16_t* h1_rel_indices = parameters.dev_rel_indices + hit_offset;

  unsigned* dev_atomics_velo = parameters.dev_atomics_velo + event_number * Velo::num_atomics;

  unsigned first_module_pair = Velo::Constants::n_module_pairs - 1;
...
...
...
``` 

- We need to have the number of events we will be analyzing, and the number of the current event we are focusing on, since events are treated in separate CUDA blocks inside Allen.

```cpp
const unsigned event_number = parameters.dev_event_list[blockIdx.x];
const unsigned number_of_events = parameters.dev_number_of_events[0];
```

- We also need pointers to the hits of the current event:

```cpp
const unsigned total_estimated_number_of_clusters =
  parameters.dev_offsets_estimated_input_size[Velo::Constants::n_module_pairs * number_of_events];
const unsigned* module_hit_start =
  parameters.dev_offsets_estimated_input_size + event_number * Velo::Constants::n_module_pairs;
const unsigned* module_hit_num = parameters.dev_module_cluster_num + event_number * Velo::Constants::n_module_pairs;
const unsigned hit_offset = module_hit_start[0];
```

- As well as pointers to the tracks of the current event:

```cpp
const unsigned tracks_offset = Velo::track_offset(parameters.dev_offsets_estimated_input_size, event_number);
Velo::TrackHits* tracks = parameters.dev_tracks + tracks_offset;
Velo::TrackletHits* three_hit_tracks = parameters.dev_three_hit_tracks +
...
```

For this, we use `Velo::track_offset` defined in `VeloEventModel.cuh`:

```cpp
...
  /**
   * @brief Returns the track offset of an event. //
   */
  __host__ __device__ inline unsigned track_offset(const unsigned* offsets, const unsigned event_number)
  {
    const auto offset_event = offsets[event_number * Velo::Constants::n_module_pairs];
    return offset_event * Velo::Constants::max_number_of_tracks_per_cluster;
  }
} // namespace Velo
```

## Looping Over VELO Clusters in Allen

First, you parallelize over the number of events by calling the CUDA kernel with a number of blocks equal to the number of events. You select the event number with:

```cpp
const unsigned event_number = parameters.dev_event_list[blockIdx.x];
```

To access the VELO clusters in an event, use `velo_cluster_container.x(index)`, etc. To find the total number of clusters in the event:

```cpp
const auto event_number_of_clusters =
  parameters.dev_offsets_estimated_input_size[(event_number + 1) * Velo::Constants::n_module_pairs] - 
  parameters.dev_offsets_estimated_input_size[event_number * Velo::Constants::n_module_pairs];
```

where `parameters.dev_offsets_estimated_input_size[event_number * Velo::Constants::n_module_pairs];` gives the number of clusters for all the events before the current event, and `parameters.dev_offsets_estimated_input_size[(event_number + 1)  * Velo::Constants::n_module_pairs];` gives the number of clusters for all the events including the "current" event.

## Notes on the [`make_velo_tracks()`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/configuration/python/AllenConf/velo_reconstruction.py#L167) Function

- File: [`velo_reconstruction.py`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/configuration/python/AllenConf/velo_reconstruction.py).
- Paper: [A fast local algorithm for track reconstruction on parallel architectures](https://ieeexplore.ieee.org/document/8778210).

Steps:

1. **Initialize** the number of events: First we initialize the number of events with `number_of_events = initialize_number_of_events()`. Implementation: [`HostInitNumberOfEvents.cpp`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/host/init_event_list/src/HostInitNumberOfEvents.cpp).

2. **Decode** VELO information: Then we decode the information from the Velo, with `decode_velo()` and store it in `decoded_velo`. See [`decode_velo()`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/configuration/python/AllenConf/velo_reconstruction.py#L18).

3. **Search by Triplet**: `velo_search_by_triplet` takes the information from the decoding, i.e. the normal clusters etc. and contructs all the tracks, using the search by triplet algorithm. Implementaiton: [`SearchByTriplet.cu`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/search_by_triplet/src/SearchByTriplet.cu).

4. **Prefix sum**: `prefix_sum_offsets_velo_tracks` is an implementation of prefix sum on the host. Implementation: [`HostPrefixSum.cpp`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/host/prefix_sum/src/HostPrefixSum.cpp).

```cpp
prefix_sum_offsets_velo_tracks = make_algorithm(
	host_prefix_sum_t,
	name="prefix_sum_offsets_velo_tracks",
	dev_input_buffer_t=velo_search_by_triplet.dev_number_of_velo_tracks_t,
)
```
Specifically, we take the variable `dev_number_of_velo_tracks`, which is simply an array containing the number of tracks we have for each event passed to Allen, and we compute the prefix sum of this array, i.e. the offsets, in order to be more computationally efficient later on.

5. **Weak track filter**: `velo_three_hit_tracks_filter` is the weak track filter algorithm, which operates on three-hit tracks,
and appends them to the final tracks container given that two
conditions are met. For more details see the [paper](https://ieeexplore.ieee.org/document/8778210).

6. **Prefix sum for three-hit tracks**: `prefix_sum_offsets_number_of_three_hit_tracks_filtered` does, exactly as above, the prefix sum on the `velo_three_hit_tracks_filter` output.

7. **Copy track hit number**: `velo_copy_track_hit_number` copies Velo track hit numbers on a consecutive container. Implementation: [`VeloCopyTrackHitNumber.cu`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/consolidate_tracks/src/VeloCopyTrackHitNumber.cu).

8. **Prefix sum for track hit number**: `prefix_sum_offsets_velo_track_hit_number` does again, exactly as above, the prefix sum on the `velo_copy_track_hit_number` output.

9. **Consolidate tracks**: Finally, `velo_consolidate_tracks` consolidates the tracks. Implementation: [`VeloConsolidateTracks.cu`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/consolidate_tracks/src/VeloConsolidateTracks.cu)

## Notes on the [Search by Triplet](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/search_by_triplet/src/SearchByTriplet.cu) Algorithm

Important files: [`VeloDefinitions.cuh`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/event_model/velo/include/VeloDefinitions.cuh), [`VeloEventModel.cuh`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/event_model/velo/include/VeloEventModel.cuh).

- In `VeloDefinitions.cuh`:

	- VELO geometry:
		1. 52 modules
		1. 26 module pairs
		1. 4 sensors per module
		1. 8 senors per module pair

	- `max_track_size = 26`

### Important Local Variables

- The `search_by_triplet` kernel is run with the configuration below.

```cpp
velo_search_by_triplet::velo_search_by_triplet<<<(7,1,1),(64,1,1)>>>
```

Meaning the events are processed in 7 independent CUDA blocks of execution, and each CUDA block has 64 threads.

- `event_number`: The number of the specific event currently being processed, e.g. here `event_number = 0`, ..., `event_number = 6`, with `-n 7`.

- `number_of_events`: The number of events passed to Allen with `-n` flag, e.g. here `number_of_events = 7`, with `-n 7`.

- `total_estimated_number_of_clusters`: The total number of clusters, across all of the events passed, e.g. for `-n 2`, and for the first event having 1,500 hits and the second 2,000, then total_estimated_number_of_clusters=3500. Here `total_estimated_number_of_clusters = 15945`.

- `module_hit_start`: The hits (15945 for `-n 7`) are stored all together. Also, the hits are stored by-module. So for 7 events, we have 26 module-pairs, hence `26*7 = 182` different `module_hit_start`. For this reason, `module_hit_start[182] = 15945`. Dereferencing past this index does not throw an error but all the values are garbage (left-over) values.

- The VELO detector has 26 module pairs.

```cpp
Velo::Constants::n_module_pairs = 26
```

- `module_hit_num`: The number of hits that each module has. Again, for 7 events, we have 26 module-pairs, hence `26*7 = 182` different `module_hit_num`. For this reason, for an `index` between 0 and 181 inclusive, you get `module_hit_num[index] ~ 100`, i.e. each module, in each event, has roughly 100 hits. Dereferencing past this index does not throw an error but all the values are garbage (left-over) values.

- `hit_offset = module_hit_start[0]`: The offset past which the hits are the hits related to the current event.

- `velo_cluster_container`: The (sorted-by-phi) container of all the Velo clusters (hits), from all the events. To initialize, we also need to pass the total number of clusters we have, here 15945, and the `hit_offset`, to be able to access the hits of the current event.

```cpp
const auto velo_cluster_container =
    Velo::ConstClusters {parameters.dev_sorted_velo_cluster_container, total_estimated_number_of_clusters, hit_offset};
```

For the type `Velo::ConstClusters` see [`VeloEventModel.cuh`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/event_model/velo/include/VeloEventModel.cuh).

- `Velo::TrackHits* tracks`: Contains the hits of all the tracks, from all the events. 

- `Velo::TrackHits`: Structure containing indices to hits within hit array. See `VeloEventModel.cuh`.

- `tracks_offsets`: The offset to reach the tracks associated with the current event.

- `hit_used`: Boolean array, where you store whether a hit has been used to form a track or not.

### Important Global Variables

- `host_number_of_events`: The number of events passed to Allen with `-n` flag, on the host.

- `dev_number_of_events`: The number of events passed to Allen with `-n` flag, on the device.

- `host_total_number_of_velo_clusters_t`: The total number of clusters, across all of the events passed.

- `dev_sorted_velo_cluster_container_t`: The (sorted-by-phi) container of all the Velo clusters (hits), from all the events.

- `dev_offsets_estimated_input_size_t`: Array containing the offsets for the clusters of each event, inside the cluster container.

- `dev_module_cluster_num_t`: Array containing the number of clusters that each module has, across all the events passed.

- `dev_number_of_velo_tracks_t`: Array containing the number of tracks we have for each event passed to Allen. So for `-n 7` this array has length of 7.

- `dev_atomics_velo`: Atomic operations/variables are used in concurrent programming to ensure the consistency and integrity of shared data structures.

In the VELO we seem to need 3 types of atomic variables:

```cpp
namespace atomics {
      enum atomic_types { number_of_three_hit_tracks, tracks_to_follow, local_number_of_hits };
    }
```

That is why `Velo::num_atomics = 3`, see `VeloDefinitions.cuh`. Hence, `dev_atomics_velo` is an array, that contains, for each event, the values for these 3 atomic variables. And [here](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/search_by_triplet/src/SearchByTriplet.cu#L433) is an example of performing one of these atomic operations (`atomicAdd`).

```cpp
const auto track_number =
        atomicAdd(dev_atomics_velo + atomics::tracks_to_follow, 1) % Velo::Constants::max_tracks_to_follow;
```

## Notes on the [`VeloCopyTrackHitNumber`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/consolidate_tracks/src/VeloCopyTrackHitNumber.cu) Algorithm

- `event_tracks`: Contains the tracks of the events, all together.

- `tracks_offset`: The offset you need to reach the tracks associated to the current event.

- `number_of_tracks`: The number of tracks in the current event.

- `accumulated_tracks`: The number of tracks in the current event, including the special three-hit tracks, from the weak track filter algorithm.

- Tracks can contain up to a maximum number of hits, specifically 26, since `max_track_size = 26`. Therefore, when I print a track,

```bash
(cuda-gdb) print event_tracks[0]          
$86 = {hits = {2641, 2546, 2442, 2340, 34, 0, 21, 0, 9, 0, 6, 0, 11, 0, 37, 0, 25, 0, 41, 0, 27, 0, 40, 0, 31, 0}, hitsNum = 4 '\004'}
```
the length of this array, is always fixed to 26. The length of the track however is `hitsNum = 4`, hence only the first 4 values are significant, viz. `{2641, 2546, 2442, 2340}`. The rest of the hits in the array are garbage values.

Another example below.

```bash
(cuda-gdb) print event_tracks[100]
$94 = {hits = {2111, 1995, 1876, 1740, 1617, 1503, 1394, 1289, 1188, 1101, 1003, 0, 1378, 0, 1371, 0, 1373, 0, 1368, 0, 1372, 0, 1374, 0, 1375, 0}, 
  hitsNum = 11 '\v'}
```

- `velo_track_hit_number`: For each track in the current event, we have an array that contains the number of hits in this track, i.e. the lengths of the track.

- `__global__ void velo_copy_track_hit_number::velo_copy_track_hit_number`: Copies Velo track hit numbers on a consecutive container.

## Notes on [`VeloConsolidateTracks.cu`](https://gitlab.cern.ch/lhcb/Allen/-/blob/master/device/velo/consolidate_tracks/src/VeloConsolidateTracks.cu)

- `event_number_of_tracks_in_main_track_container`: The number of tracks, from the search by triplet algorithm, for the current event.

- `event_number_of_three_hit_tracks_filtered`: The number of tracks, from the weak track filter, for the current event.

- `event_total_number_of_tracks =` `event_number_of_tracks_in_main_track_container +` `event_number_of_three_hit_tracks_filtered`
