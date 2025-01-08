---
title: remotion视频渲染
categories:
  - 轮子应用
excerpt: '通过remotion视频做一些视频渲染的能力'
description: '通过remotion视频做一些视频渲染的能力'
date: 2025-01-08 16:06:58
---


## remotion

官网： [https://www.remotion.dev/](https://www.remotion.dev/)


## 绿幕渲染

VideoProcessor.tsx
```tsx
import { useCallback, useRef } from "react";
import { AbsoluteFill, OffthreadVideo, useVideoConfig } from "remotion";

export const Greenscreen: React.FC<{
  opacity: number; // 透明度参数
  src: string;
}> = ({ opacity, src }) => {
  const canvas = useRef<HTMLCanvasElement>(null);
  const { width, height } = useVideoConfig();

  // 处理每一帧
  const onVideoFrame = useCallback(
    (frame: CanvasImageSource) => {
      if (!canvas.current) {
        return;
      }
      const context = canvas.current.getContext('2d');

      if (!context) {
        return;
      }

      context.drawImage(frame, 0, 0, width, height);
      const imageFrame = context.getImageData(0, 0, width, height);
      const { length } = imageFrame.data;

      // 如果像素非常绿色，减少 alpha 通道
      // 每次迭代步长为4，因为每个像素由四个值（红、绿、蓝、透明度）组成。
      for (let i = 0; i < length; i += 4) {
        const red = imageFrame.data[i + 0];
        const green = imageFrame.data[i + 1];
        const blue = imageFrame.data[i + 2];
        if (green > 100 && red < 100 && blue < 100) {
          imageFrame.data[i + 3] = opacity * 255; // 设置透明度
        }
      }
      context.putImageData(imageFrame, 0, 0); // 更新画布
    },
    [height, width],
  );

  return (
    <AbsoluteFill>
      <AbsoluteFill>
        <OffthreadVideo
          style={{ opacity: 0 }}
          onVideoFrame={onVideoFrame}
          src={src}
        />
      </AbsoluteFill>
      <AbsoluteFill>
        <canvas ref={canvas} width={width} height={height} />
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```


Root.tsx
```tsx
import './tailwind.css';
import { AbsoluteFill, Composition, Img, Sequence, staticFile } from "remotion";
import { Greenscreen } from "./VideoProcessor";

const ChromaKeyComposition: React.FC = () => {

  return (
    <AbsoluteFill>
      {/* Background Image */}
      <Sequence from={0}>
        <AbsoluteFill>
          <Img
            src={staticFile("1.png")}
            style={{
              width: '100%',
              height: '100%',
              objectFit: 'cover',
            }}
          />
        </AbsoluteFill>
      </Sequence>

      {/* Green Screen Video */}
      <Sequence from={0}>
        <AbsoluteFill
          style={{
            display: 'flex',
            justifyContent: 'center',
            alignItems: 'center',
          }}
        >
          <Greenscreen
            opacity={0}
            src={staticFile("a.mp4")}
          />
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};


export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="HelloWorld"
        component={ChromaKeyComposition}
        durationInFrames={1500}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{}}
      />
    </>
  );
};
```