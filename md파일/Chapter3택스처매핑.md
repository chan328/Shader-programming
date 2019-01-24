**Part 3 택스처 매핑**
=========================

- 앞에서 3D 물체를 이루는 구성요소는 삼각형이며, 정점 3개로 삼각형을 만들 수 있다고 하였다. <br>
그렇다면 그 삼각형 위에 이미지를 입히기 위해서는 어떻게 해야하는가?

삼각형은 정점 3개로 이루어져 있으므로 각 정점을 택스처 위에 있는 한 픽셀에 대응시켜 주면 된다. <br>

- 그럼 택스처 위에 한 픽셀을 어떻게 가리키는가? 
 
 텍스쳐도 결국 이미지 파일이므로 `x = 0` 이 텍스처의 제일 `왼쪽 열`을, `x = 1` 은 `가장 오른쪽 열`을 나타낸다고 표현하면 된다. 마찬가지로 y좌표도 `y = 0`이 택스처의 제일 `위쪽 열`을 `y = 0`은 택스처의 제일 아래 열을 나타낸다.

 ![Texture](image\4.png)


 정점셰이더의 입력데이터
----------------------

```
 struct VS_INPUT
 {
    float4 mPosition : POSITION;
    float2 mTexCoord : TEXCOORD0;
 };
 ```

UV좌표는 u와 v로 나눠지므로 데이터형은 float2가 됩니다. float4 mPosition과 같이 mTexCoord도 자신만의 시맨틱을 가지게 되는데 이때 `TEXCOORD (택스처 좌표의 줄임말)`라는 시맨틱이 mTexCoord의 시맨틱입니다.


정점셰이더의 출력데이터
--------------------

```
struct VS_OUTPUT
{
    float4 mPosition : POSTION;
    float2 mTexCoord : TEXCOORD0;
};
```

정점셰이더는 위치정보 외에도 다른 정보들을 반환할 수 있는데 이 외의 정보를 추가하는것은 레스터라이저를 위해서가 아닌 픽셀 셰이더를 위해서입니다.

정점셰이더 함수
--------------
```
VS_OUTPUT vs_main( VS_INPUT Input )
{
    VS_OUTPUT Output;

    Output.mPosition = mul(Input.mPosition, gWorldMatrix);
    Output.mPosition = mul(Output.mPosition, gViewMatrix);
    Output.mPosition = mul(Output.mPosition, gProjectionMatrix);
```
UV 좌표를 전달할 차례인데, Output 구조체에 UV 좌표를 대입하기 전에 공간변환은 할 필요가 없습니다.<br> UV좌표는 3차원 공간상에 존재하지 않기 때문에 변환없이 그대로 넘겨주면 됩니다.
```
    Output.mTexCoord = Input.mTexCoord;

    return Output;
}
```

픽셀 셰이더의 입력데이터 및 전역변수
----------------------------------
- 픽셀 셰이더에서 할 일은 택스처 이미지에서 텍셀을 구해 그 색을 화면에 출력하는 것입니다.<br>
따라서 택스처로 사용할 이미지와 현재 픽셀의 UV 좌표가 필요하게 됩니다. 택스처 이미지는 픽셀마다 변하는 값이 아니므로 전역변수로, <br>
UV 좌표는 정점 셰이더로부터 보간기를 거쳐 들어온 입력데이터가 됩니다.

```
struct VS_INPUT
{
    float2 mTexCoord : TEXCOORD0;
```
이제 택스처를 선언해야 합니다. RenderMonkey 프로젝트를 설정할 때 DiffuseSampler란 이름의 택스처 개체를 만들었는데, 이 개체가 택셀을 구할 때 사용되는 택스처 샘플러입니다.
```
    Sampler2D DiffuseSampler;
};
```
 
 픽셀 셰이더
 ----------
 ```
 float4 ps_main (PS_INPUT Input) : COLOR
 {
    float4 albedo = tex2D(DiffuseSampler, Input.mTexCoord);

    return albedo;
 }
 ```