# Iteration 11. Pseudo beam-search

_25-08-2024_

## Goal

Can I improve the accuracy of the predictions by using a pseudo-beam-search?

## Motivation

Beam-search has been probed to generate more accurate responses than greedy decoding.
However it is not efficiently implemented on VLLM.

My idea is to generate n responses for the same prompt and select the one with the highest
logprob. This would be similar to beam-search, but the implementation would be much more efficient.

## Development

## Results

### Inference speed

| n  | runtime | estimated runtime (min) |
|----|---------|-------------------------|
| 1  | 1m50    | 1.8                     |
| 2  | 2m32    | 2.5                     |
| 4  | 3m55    | 3.8                     |
| 8  | 6m32    | 6.5                     |
| 16 | 11m46   | 11.8                    |
| 32 | 22m43   | 22.5                    |

The runtime increases linearly with `n`, however there is a constant time that makes that using `n=4` only twice the time as `n=1`.

```bash
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --temperature=0.1 --output_filepath=submission_qwen05_x8_T01_n1.json --n=1
```

This table could work as a rule of thumb of the slowdown when using pseudo beam-search.

| n  | slowdown |
|----|----------|
| 1  | 1        |
| 4  | 2        |
| 10 | 4        |
| 20 | 8        |

### Accuracy effect

On a previous iteration I was able to see improvements due to beam-search with just 8 predictions per task. Let's try do to the same.
I will be using `n=20` and different temperatures.

![8 predictions](res/2024-08-26-05-10-51.png)

![16 predictions](res/2024-08-26-05-10-58.png)

The tendency when increasing the temperature is completely different to the observed on previous experiments. But the improvement in accuracy is not clear.

```bash
# estimated runtime 1h30 ~ 16*6
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T01.json --temperature=0.1
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T02.json --temperature=0.2
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T04.json --temperature=0.4
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T06.json --temperature=0.6
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T08.json --temperature=0.8
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=8 --n=20 --output_filepath=submission_qwen05_x8_n20_T10.json --temperature=1.0

python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T01.json --temperature=0.1
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T02.json --temperature=0.2
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T04.json --temperature=0.4
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T06.json --temperature=0.6
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T08.json --temperature=0.8
python inference.py --model_path=/home/gbarbadillo/data/Qwen2-0.5B-arc --predictions_per_task=16 --n=20 --output_filepath=submission_qwen05_x16_n20_T10.json --temperature=1.0
```

#### Full experiment

I have run an experiment for 4h49: `submission_qwen05_x128_n20_T08` and I'm comparing it to `submission_qwen15_x128`.
The accuracy improves from 5% to 5.2% so the improvement is tiny, and the inference time has been close to 10 times.
So it does not seem this is a promising path.

## Conclusion

Accuracy improvement is tiny despite inference time being increased a lot. Is better to use compute in other ways.

## Next steps

## TODO

- [x] Modify inference script to support this
  - [x] Are the outputs provided by LLM sorted by logprob, or I have to sort them myself? YES THEY ARE ALREADY SORTED
- [x] How does the inference speed changes when requesting more responses per prompt?
- [x] Does the accuracy improves?
