# DeiT Paper Reproduction(2021)

## Introduction
This project reproduces the original DeiT architecture as described in the original paper. The purpose of this project is to check if DeiT helps ViT architecture converge faster.

## Architecture Summary
| Stage                   | Layer                | Type                | Parameters                    | Output Shape  |
| ----------------------- | -------------------- | ------------------- | ----------------------------- | ------------- |
| Input                   | Input Image          | RGB Image           | –                             | 224 × 224 × 3 |
| **Patch Embedding**     | Conv2D               | Patch Projection    | 384 filters, 16×16, stride 16 | 14 × 14 × 384 |
|                         | Reshape              | Flatten Patches     | –                             | 196 × 384     |
| **Embedding**           | CLS Token            | Learnable Token     | 1 × 384                       | 1 × 384       |
|                         | Distill Token        | Learnable Token     | 1 × 384                       | 1 × 384       |
|                         | Concat               | Token Concatenation | CLS + Distill + Patches       | 198 × 384     |
|                         | Positional Embedding | Learnable           | 198 × 384                     | 198 × 384     |
|                         | Dropout              | Dropout             | p = 0.0                       | 198 × 384     |
| **Transformer Encoder** | Encoder Block ×12    | MHSA + MLP          | 6 heads, MLP dim = 1536       | 198 × 384     |
|                         | LayerNorm            | Final Norm          | –                             | 198 × 384     |
| **Classification Head** | CLS Head             | Dense               | 384 → #classes                | #classes      |
| **Distillation Head**   | Distill Head         | Dense               | 384 → #classes                | #classes      |
| **Output**              | Ensemble             | Avg / Distill Loss  | –                             | #classes      |

## Dataset
tiny-imagenet-200 is used as the dataset in this reproduction. The images are argumented aligns with the original paper (random resized crop, random horizontal flip, cutmix, mixup, erase, autoaugment and label smoothing). We only use training set in this reproduction, as our purpose is to check if DeiT converges faster than ViT, not training a DeiT for good performance.

## Teacher model
We use pre-trained ViT(google/vit-base-patch16-224-in21k) as the teacher model. For better performance, we fine-tune it with  tiny-imagenet-200 before using (only train the classification head for 5 epochs then train the whole model for 10 epochs)
One of the reasons for using ViT not traditional CNN as teacher model
is that they are both ViTs, which reduces the inductive gap between them. Moreover, it can help DeiT focus more on knowledge, instead of solving mismatch in architecture.

## Results
The training loss(0.5 weight for cls loss and 0.5 for dis loss) for 40 epochs for DeiT is below

We also attach the training accuracy graph for 40 epochs for ViT, which is below. For better comparison, the loss for ViT decreases from 5.62 to 5.29 after 40 epochs and stays around 5.3 for 37 epochs


## Discussion
Compared with ViT, we can see that DeiT converges significantly faster than ViT(from 5.39 to 3.62 within 40 epochs), while ViT seems to be stuck at 5.3, which is expected, as its inductive bias is low, according to Dosovitskiy et al. (2020). The initial gradients are noisy and the initial representations are unstructured for ViT. With limited data like tiny-imagenet-200, this make optimisation harder. DeiT introduces teacher model, which gives it additional supervision, not pure data. Consequently, DeiT can align with teacher's behaviour, not finding patterns from scratch, which helps it converge faster.

## References
Baevski, A., Auli, M., & Grangier, D. (2021).
Training data-efficient image transformers & distillation through attention.
Proceedings of the 38th International Conference on Machine Learning (ICML 2021), PMLR 139, 10347–10357.
https://arxiv.org/abs/2012.12877

Dosovitskiy, A., Beyer, L., Kolesnikov, A., Weissenborn, D., Zhai, X., Unterthiner, T., Dehghani, M., Minderer, M., Heigold, G., Gelly, S., Uszkoreit, J., & Houlsby, N. (2020). An image is worth 16×16 words: Transformers for image recognition at scale. *arXiv*. https://arxiv.org/abs/2010.11929

Hugging Face. (n.d.). google/vit-base-patch16-224-in21k [Model]. Hugging Face. Retrieved January 22, 2026, from https://huggingface.co/google/vit-base-patch16-224-in21k

Tiny ImageNet Dataset
Wu, J., Zhang, J., Xie, Y., & others. (2017).
Tiny ImageNet Visual Recognition Challenge.
Stanford University.
https://tiny-imagenet.herokuapp.com/
