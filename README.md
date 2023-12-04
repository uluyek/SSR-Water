# SSR-Water

### Attempt on Shaders: 

I made several attempts on Opaque shaders that eveutally fail to show up. This is one of the attempts. Eyad's shader graph worked well, but we can't use shadergraph in the actual game. 
```csharp
Shader "Custom/Water_Opaque"
{
    Properties
    {
        _MainTex("Albedo (RGB)", 2D) = "white" {}
        _Color("Tint Color", Color) = (1,1,1,1) // Default to white
        _Glossiness("Smoothness", Range(0,1)) = 0.5
        _Metallic("Metallic", Range(0,1)) = 0.0
        _SSRReflectionTex("SSR Reflection Texture", 2D) = "white" {} // SSR reflection texture
        _ReflectedColorMap("Reflected Color Map", 2D) = "black" {} // Joshua's reflected color map
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry" }

        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows
        #pragma target 3.0

        sampler2D _MainTex;
        float4 _Color;
        sampler2D _SSRReflectionTex;
        sampler2D _ReflectedColorMap; // Adding Joshua's reflected color map
        float _Glossiness;
        float _Metallic;

        struct Input
        {
            float2 uv_MainTex;
            float3 worldPos; // World position for SSR UV calculation
            float3 worldRefl; // World reflection vector for SSR
        };

        float4 WorldToScreenPos(float3 worldPos) {
            float4 clipSpacePos = mul(UNITY_MATRIX_VP, float4(worldPos, 1.0));
            return clipSpacePos / clipSpacePos.w;
        }

        float2 ComputeReflectionUV(float3 worldPos, float3 worldNormal, float3 viewDir) {
            float3 reflection = reflect(-viewDir, worldNormal);
            float3 reflectionWorldPos = worldPos + reflection;
            float4 screenPos = WorldToScreenPos(reflectionWorldPos);
            return screenPos.xy * 0.5 + 0.5;
        }

        void surf (Input IN, inout SurfaceOutputStandard o) {
            fixed4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
            o.Alpha = c.a;

            float3 viewDir = normalize(_WorldSpaceCameraPos - IN.worldPos);
            float2 ssrUV = ComputeReflectionUV(IN.worldPos, IN.worldRefl, viewDir);

            fixed4 ssrColor = tex2D(_SSRReflectionTex, ssrUV);
            fixed4 reflectedColor = tex2D(_ReflectedColorMap, ssrUV); // Sampling Joshua's reflected color map

            // Blend SSR reflection with albedo
            fixed4 finalColor = lerp(ssrColor, reflectedColor, _Glossiness); // Blend with glossiness as a factor
            o.Emission = lerp(o.Albedo, finalColor.rgb, _Glossiness);
        }
        ENDCG
    }
    FallBack "Diffuse"
}

```
