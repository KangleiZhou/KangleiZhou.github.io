---
layout: post
category: HoloLens 2
title: 【AR/VR】HoloLens 2 demo
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - HoloLens 2
published: true
---







# Demo

[Simple App of Microsoft HoloLens 2](https://www.bilibili.com/video/BV1xK4y1d7sW)

# C# 代码

```C#
using System;
using System.Collections;
using System.Collections.Generic;

using UnityEngine;

using Microsoft;
using Microsoft.MixedReality.Toolkit.Utilities;
using Microsoft.MixedReality.Toolkit.Input;


public class HandTracking : MonoBehaviour
{
    public int jointsNum = 26;
    public GameObject jointObject;
    public Vector3 scale = new Vector3(0.01f, 0.01f, 0.01f);
    public Color[] jointsColor = new Color[26] {
        Color.yellow, Color.clear, Color.magenta, Color.magenta, Color.magenta,
        Color.magenta, Color.cyan, Color.cyan, Color.cyan, Color.cyan,
        Color.cyan, Color.red, Color.red, Color.red, Color.red,
        Color.red, Color.green, Color.green, Color.green, Color.green,
        Color.green, Color.blue, Color.blue, Color.blue, Color.blue, Color.blue
    };

    public GameObject line;
    public float lineWidth = 0.005f;
    public int fingersNum = 5;
    public int[,] fingers = new int[5, 6] {
        {0, 2, 3, 4, 5, 5},
        {0, 6, 7, 8, 9, 10},
        {0, 11, 12, 13, 14, 15},
        {0, 16, 17, 18, 19, 20},
        {0, 21, 22, 23, 24, 25}
    };
    public Color[] fingersColor = new Color[5] {
        Color.magenta, Color.cyan, Color.red, Color.green, Color.blue
    };

    public float minDist = 0.02f;
    public float tmpDistL = 1f;
    public float tmpDistR = 1f;

    Vector3[] jointsL = new Vector3[26];
    Vector3[] jointsR = new Vector3[26];

    List<GameObject> jointObjectsL = new List<GameObject>();
    List<GameObject> jointObjectsR = new List<GameObject>();

    List<LineRenderer> fingerBonesL = new List<LineRenderer>();
    List<LineRenderer> fingerBonesR = new List<LineRenderer>();

    List<LineRenderer> lrWords = new List<LineRenderer>();

    MixedRealityPose pose;
    

    // Start is called before the first frame update
    void Start()
    {
        // initialization
        for (int i = 0; i < jointsNum; i++)
        {
            GameObject obj1 = Instantiate(jointObject, this.transform);
            jointObjectsL.Add(obj1);

            GameObject obj2 = Instantiate(jointObject, this.transform);
            jointObjectsR.Add(obj2);
        }

        for (int i = 0; i < fingersNum; i++)
        {
            LineRenderer lr1 = InitialLine(fingersColor[i], fingersColor[i], lineWidth, lineWidth, 6);
            fingerBonesL.Add(lr1);

            LineRenderer lr2 = InitialLine(fingersColor[i], fingersColor[i], lineWidth, lineWidth, 6);
            fingerBonesR.Add(lr2);
        }
    }

    // Update is called once per frame
    void Update()
    {
        // only render if hand is tracked
        for (int i = 0; i < jointsNum; i++)
        {
            jointObjectsL[i].GetComponent<Renderer>().enabled = false;
            jointObjectsR[i].GetComponent<Renderer>().enabled = false;
        }

        for (int i = 0; i < fingersNum; i++)
        {
            fingerBonesL[i].enabled = false;
            fingerBonesR[i].enabled = false;
        }

        // obtain all the joints of left and right hands respectively
        for (int i = 0; i < jointsNum; i++)
        {
            if (HandJointUtils.TryGetJointPose((TrackedHandJoint)(i + 1), Handedness.Left, out pose))
            {
                jointsL[i] = pose.Position;
                jointObjectsL[i].GetComponent<Renderer>().enabled = true;

                //Debug.Log("Left hand, joint - " + (i + 1) + " "
                //    + (TrackedHandJoint)(i + 1)
                //    + ": " + jointsL[i]);
            }

            if (HandJointUtils.TryGetJointPose((TrackedHandJoint)(i + 1), Handedness.Right, out pose))
            {
                jointsR[i] = pose.Position;
                jointObjectsR[i].GetComponent<Renderer>().enabled = true;

                //Debug.Log("Right hand, joint - " + (i + 1) + " "
                //    + (TrackedHandJoint)(i + 1)
                //    + ": " + jointsR[i]);
            }

        }

        // set joints position
        for (int i = 0; i < jointsNum; i++)
        {
            jointObjectsL[i].transform.position = jointsL[i];
            jointObjectsL[i].GetComponent<Renderer>().material.color = jointsColor[i];
            jointObjectsL[i].transform.localScale = new Vector3(0.01f, 0.01f, 0.01f);

            jointObjectsR[i].transform.position = jointsR[i];
            jointObjectsR[i].GetComponent<Renderer>().material.color = jointsColor[i];
            jointObjectsR[i].transform.localScale = new Vector3(0.01f, 0.01f, 0.01f);
        }

        // draw bone line
        for (int i = 0; i < fingersNum; i++)
        {
            for (int j = 0; j < 6; j++)
            {
                if (jointObjectsL[0].GetComponent<Renderer>().enabled)
                {
                    fingerBonesL[i].enabled = true;
                    fingerBonesL[i].SetPosition(j, jointsL[fingers[i, j]]);
                }

                if (jointObjectsR[0].GetComponent<Renderer>().enabled)
                {
                    fingerBonesR[i].enabled = true;
                    fingerBonesR[i].SetPosition(j, jointsR[fingers[i, j]]);
                }
            }
        }

        // judge wirting characters
        DrawLine(jointsL[5], jointsL[10], true);
        DrawLine(jointsR[5], jointsR[10], false);

        // judge whlether clear all paintings
        IsClear(jointsR[5], jointsR[15]);
        IsClear(jointsL[5], jointsL[15]);
    }
    
    void IsClear(Vector3 thumbTip, Vector3 middleTip)
    {
        float distThumbTip2MiddleTip = Vector3.Distance(thumbTip, middleTip);

        if (distThumbTip2MiddleTip < minDist && distThumbTip2MiddleTip != 0)
        {
            foreach (LineRenderer lr in lrWords)
            {
                lr.enabled = false;
            }
            
            Debug.Log(distThumbTip2MiddleTip + "-------CLEAR ALL LINE #" + lrWords.Count + "--------");
        }
    }

    void DrawLine(Vector3 indexTip, Vector3 thumbTip, bool isLeft)
    {
        float distIndexTip2ThumbTip = Vector3.Distance(indexTip, thumbTip);

        if (distIndexTip2ThumbTip < minDist && distIndexTip2ThumbTip != 0)
        {
            // go to draw a word
            if ((tmpDistL > minDist && isLeft) || (tmpDistR > minDist && !isLeft)) // establish a LineRenderer
            {
                Color randomLineColor = new Color(
                    UnityEngine.Random.Range(0f, 1f),
                    UnityEngine.Random.Range(0f, 1f),
                    UnityEngine.Random.Range(0f, 1f)
                );
                LineRenderer lr = InitialLine(randomLineColor, randomLineColor, lineWidth * 2.0f, lineWidth * 2.0f, 0);
                lrWords.Add(lr);
                Debug.Log(distIndexTip2ThumbTip + "-------CREATE LINE #" + lrWords.Count + "--------" + isLeft);
            }
            else
            {
                LineRenderer lr = lrWords[lrWords.Count - 1];
                lr.positionCount++;
                lr.SetPosition(lr.positionCount - 1, indexTip);

                Debug.Log(lr.positionCount + "-------DRAW LINE #" + lrWords.Count + " POINT #" + lrWords.Count + "--------");
            }  
        }

        if (isLeft)
        {
            tmpDistL = distIndexTip2ThumbTip;
        }
        else
        {
            tmpDistR = distIndexTip2ThumbTip;
        }
    }

    LineRenderer InitialLine(Color startColor, Color endColor, float startWidth, float endWidth, int positionCount)
    {
        GameObject obj = Instantiate(line, this.transform);
        LineRenderer lr = obj.GetComponent<LineRenderer>();
        lr.positionCount = positionCount;
        lr.startColor = startColor;
        lr.endColor = endColor;
        lr.startWidth = startWidth;
        lr.endWidth = endWidth;

        return lr;
    }
}


//IndexDistalJoint 10
//The joint nearest the tip of the index finger.

//IndexKnuckle	8	
//The knuckle joint of the index finger.

//IndexMetacarpal	7	
//The lowest joint of the index finger.

//IndexMiddleJoint	9	
//The middle joint of the index finger.

//IndexTip	11	
//The tip of the index finger.

//MiddleDistalJoint	15	
//The joint nearest the tip of the finger.

//MiddleKnuckle	13	
//The knuckle joint of the middle finger.

//MiddleMetacarpal	12	
//The lowest joint of the middle finger.

//MiddleMiddleJoint	14	
//The middle joint of the middle finger.

//MiddleTip	16	
//The tip of the middle finger.

//None	0	
//Palm	2	
//The palm.

//PinkyDistalJoint	25	
//The joint nearest the tip of the pink finger.

//PinkyKnuckle	23	
//The knuckle joint of the pinky finger.

//PinkyMetacarpal	22	
//The lowest joint of the pinky finger.

//PinkyMiddleJoint	24	
//The middle joint of the pinky finger.

//PinkyTip	26	
//The tip of the pinky.

//RingDistalJoint	20	
//The joint nearest the tip of the ring finger.

//RingKnuckle	18	
//The knuckle of the ring finger.

//RingMetacarpal	17	
//The lowest joint of the ring finger.

//RingMiddleJoint	19	
//The middle joint of the ring finger.

//RingTip	21	
//The tip of the ring finger.

//ThumbDistalJoint	5	
//The thumb's first (furthest) joint.

//ThumbMetacarpalJoint	3	
//The lowest joint in the thumb (down in your palm).

//ThumbProximalJoint 4
//The thumb's second (middle-ish) joint.

//ThumbTip	6	
//The tip of the thumb.

//Wrist	1	
//The wrist.
```

