---
layout: post
title: A solution of SHA 512
comments: true
author: "Yin Haomin"
keywords: SHA 512
---

这是我大学毕业那年写的SHA 512的一个C#的实现，我看到我的github上有人star了这个解决方案，我猜有人打算抄我的代码作为课程设计或者毕业设计了。:)
SHA (Secure Hash Algorithm，译作安全散列算法) 是美国国家安全局 (NSA) 设计，美国国家标准与技术研究院 (NIST) 发布的一系列密码散列函数。
SHA-512 (这些有时候也被称做 SHA-2)。

```
﻿using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
//using System.Convert;
 
namespace SHA512
{
    //Tested
    public class Merge
    {
        //将8个byte数合并为一个ulong型数据
        //Tested
        public static ulong BytToUlon(byte []bytearray,int i)
        {
            ulong rtnlon;
            ulong rtnlon1 = bytearray[i];
            ulong rtnlon2 = bytearray[i+1];
            ulong rtnlon3 = bytearray[i+2];
            ulong rtnlon4 = bytearray[i+3];
            ulong rtnlon5 = bytearray[i+4];
            ulong rtnlon6 = bytearray[i+5];
            ulong rtnlon7 = bytearray[i+6];
            ulong rtnlon8 = bytearray[i+7];
            rtnlon1 = RingShift.AbsoluteBit(rtnlon1 << 56, 56, 2);
            rtnlon2 = RingShift.AbsoluteBit(rtnlon2 << 48, 48, 2);
            rtnlon3 = RingShift.AbsoluteBit(rtnlon3 << 40, 40, 2);
            rtnlon4 = RingShift.AbsoluteBit(rtnlon4 << 32, 32, 2);
            rtnlon5 = RingShift.AbsoluteBit(rtnlon5 << 24, 24, 2);
            rtnlon6 = RingShift.AbsoluteBit(rtnlon6 << 16, 16, 2);
            rtnlon7 = RingShift.AbsoluteBit(rtnlon7 << 8, 8, 2);
            rtnlon8 = RingShift.AbsoluteBit(rtnlon8 , 0, 2);
            rtnlon = rtnlon1 | rtnlon2 | rtnlon3 | rtnlon4 | rtnlon5 | rtnlon6 | rtnlon7 | rtnlon8;
            return rtnlon;
        }        //将8个64bit数据合并为512位数据
        //Tested
        public static string MergeResult(ulong[] H)
        {
            string str="";
            int i = 0;
            while (i < 8)
            {
                str = str+H[i];
                i++;
            }            
            return str;
        }
    }
    //Tested
    public class BoxTransfer
    {
        public static ulong []K={
        (0x428a2f98d728ae22),  (0x7137449123ef65cd),
        (0xb5c0fbcfec4d3b2f),  (0xe9b5dba58189dbbc),
        (0x3956c25bf348b538),  (0x59f111f1b605d019),
        (0x923f82a4af194f9b),  (0xab1c5ed5da6d8118),
        (0xd807aa98a3030242),  (0x12835b0145706fbe),
        (0x243185be4ee4b28c),  (0x550c7dc3d5ffb4e2), 
        (0x72be5d74f27b896f),  (0x80deb1fe3b1696b1),
        (0x9bdc06a725c71235),  (0xc19bf174cf692694), 
        (0xe49b69c19ef14ad2),  (0xefbe4786384f25e3), 
        (0x0fc19dc68b8cd5b5),  (0x240ca1cc77ac9c65), 
        (0x2de92c6f592b0275),  (0x4a7484aa6ea6e483), 
        (0x5cb0a9dcbd41fbd4),  (0x76f988da831153b5),
        (0x983e5152ee66dfab),  (0xa831c66d2db43210),
        (0xb00327c898fb213f),  (0xbf597fc7beef0ee4), 
        (0xc6e00bf33da88fc2),  (0xd5a79147930aa725), 
        (0x06ca6351e003826f),  (0x142929670a0e6e70), 
        (0x27b70a8546d22ffc),  (0x2e1b21385c26c926), 
        (0x4d2c6dfc5ac42aed),  (0x53380d139d95b3df), 
        (0x650a73548baf63de),  (0x766a0abb3c77b2a8), 
        (0x81c2c92e47edaee6),  (0x92722c851482353b), 
        (0xa2bfe8a14cf10364),  (0xa81a664bbc423001), 
        (0xc24b8b70d0f89791),  (0xc76c51a30654be30), 
        (0xd192e819d6ef5218),  (0xd69906245565a910), 
        (0xf40e35855771202a),  (0x106aa07032bbd1b8), 
        (0x19a4c116b8d2d0c8),  (0x1e376c085141ab53), 
        (0x2748774cdf8eeb99),  (0x34b0bcb5e19b48a8), 
        (0x391c0cb3c5c95a63),  (0x4ed8aa4ae3418acb), 
        (0x5b9cca4f7763e373),  (0x682e6ff3d6b2b8a3), 
        (0x748f82ee5defb2fc),  (0x78a5636f43172f60), 
        (0x84c87814a1f0ab72),  (0x8cc702081a6439ec), 
        (0x90befffa23631e28),  (0xa4506cebde82bde9), 
        (0xbef9a3f7b2c67915),  (0xc67178f2e372532b), 
        (0xca273eceea26619c),  (0xd186b8c721c0c207), 
        (0xeada7dd6cde0eb1e),  (0xf57d4f7fee6ed178), 
        (0x06f067aa72176fba),  (0x0a637dc5a2c898a6), 
        (0x113f9804bef90dae),  (0x1b710b35131c471b),
        (0x28db77f523047d84),  (0x32caab7b40c72493), 
        (0x3c9ebe0a15c9bebc),  (0x431d67c49c100d4c), 
        (0x4cc5d4becb3e42b6),  (0x597f299cfc657e2a), 
        (0x5fcb6fab3ad6faec),  (0x6c44198c4a475817)};

        public static ulong[] H = {
        (0x6a09e667f3bcc908),  (0xbb67ae8584caa73b),
        (0x3c6ef372fe94f82b),  (0xa54ff53a5f1d36f1), 
        (0x510e527fade682d1),  (0x9b05688c2b3e6c1f), 
        (0x1f83d9abfb41bd6b),  (0x5be0cd19137e2179)};
        public static ulong[] H2 = {
        (0x6a09e667f3bcc908),  (0xbb67ae8584caa73b),
        (0x3c6ef372fe94f82b),  (0xa54ff53a5f1d36f1), 
        (0x510e527fade682d1),  (0x9b05688c2b3e6c1f), 
        (0x1f83d9abfb41bd6b),  (0x5be0cd19137e2179)};
    }
    //Tested
    public class RingShift
    {        //校正64位数中由于内存原因空位不为0的情况
        //i=1时，左移，i=0时，右移
        //Tested
        public static ulong AbsoluteBit(ulong x, int n,int i)
        {
            ulong aboBit;

            //左移的状况
            //First Ring Shift ajust
            if (i == 0)
            {
                if (n == 28)
                {
                    aboBit = x & 0xfffffff000000000;
                    return aboBit;
                }
                if (n == 34)
                {
                    aboBit = x & 0xffffffffc0000000;
                    return aboBit;
                }
                if (n == 39)
                {
                    aboBit = x & 0xfffffffffe000000;
                    return aboBit;
                }

                //Second Ring Shift ajust
                if (n == 14)
                {
                    aboBit = x & 0xfffc000000000000;
                    return aboBit;
                }
                if (n == 18)
                {
                    aboBit = x & 0xffffc00000000000;
                    return aboBit;
                }
                if (n == 41)
                {
                    aboBit = x & 0xffffffffff800000;
                    return aboBit;
                }

                //Third Ring Shift ajust
                if (n == 1)
                {
                    aboBit = x & 0x8000000000000000;
                    return aboBit;
                }
                if (n == 8)
                {
                    aboBit = x & 0xff00000000000000;
                    return aboBit;
                }
                if (n == 7)
                {
                    aboBit = x & 0xfe00000000000000;
                    return aboBit;
                }
                
                //Fourth Ring Shift ajust
                if (n == 19)
                {
                    aboBit = x & 0xffffe00000000000;
                    return aboBit;
                }
                if (n == 61)
                {
                    aboBit = x & 0xfffffffffffffff8;
                    return aboBit;
                }
                if (n == 6)
                {
                    aboBit = x & 0xfc00000000000000;
                    return aboBit;
                }
                else return 0x0000000000000000;
            }
                
                //右移的状况
            if (i == 1)
            {
                //First Ring Shift ajust
                if (n == 36)
                {
                    aboBit = x & 0x0000000fffffffff;
                    return aboBit;
                }
                if (n == 30)
                {
                    aboBit = x & 0x000000003fffffff;
                    return aboBit;
                }
                if (n == 25)
                {
                    aboBit = x & 0x0000000001ffffff;
                    return aboBit;
                }

                //Second Ring Shift ajust
                if (n == 50)
                {
                    aboBit = x & 0x0003ffffffffffff;
                    return aboBit;
                }
                if (n == 46)
                {
                    aboBit = x & 0x0000cfffffffffff;
                    return aboBit;
                }
                if (n == 23)
                {
                    aboBit = x & 0x000000000007ffff;
                    return aboBit;
                }

                //Third Ring Shift ajust
                if (n == 63)
                {
                    aboBit = x & 0x7fffffffffffffff;
                    return aboBit;
                }
                if (n == 56)
                {
                    aboBit = x & 0x00ffffffffffffff;
                    return aboBit;
                }
                if (n == 57)
                {
                    aboBit = x & 0x01ffffffffffffff;
                    return aboBit;
                }

                //Fourth Ring Shift ajust
                if (n == 45)
                {
                    aboBit = x & 0x00001fffffffffff;
                    return aboBit;
                }
                if (n == 3)
                {
                    aboBit = x & 0x0000000000000007;
                    return aboBit;
                }
                if (n == 58)
                {
                    aboBit = x & 0x03ffffffffffffff;
                    return aboBit;
                }
                else return 0x0000000000000000;
            }
            else 
            {
                if (n==56)
                {
                    aboBit = x & 0xff00000000000000;
                    return aboBit;
                }
                if (n == 48)
                {
                    aboBit = x & 0x00ff000000000000;
                    return aboBit;
                }
                if (n == 40)
                {
                    aboBit = x & 0x0000ff0000000000;
                    return aboBit;
                }
                if (n == 32)
                {
                    aboBit = x & 0x000000ff00000000;
                    return aboBit;
                }
                if (n == 24)
                {
                    aboBit = x & 0xff000000ff000000;
                    return aboBit;
                }
                if (n == 16)
                {
                    aboBit = x & 0xff00000000ff0000;
                    return aboBit;
                }
                if (n == 8)
                {
                    aboBit = x & 0xff0000000000ff00;
                    return aboBit;
                }
                if (n == 0)
                {
                    aboBit = x & 0xff000000000000ff;
                    return aboBit;
                }
                return 0;
            }
        }

        //实现移位，并实现校正64位数中由于内存原因空位不为0的情况
        public static ulong CyclicShift(ulong x,int n)
        {
            ulong cycshft = (AbsoluteBit(x << n, n, 0)) | (AbsoluteBit(x >> (64 - n), (64 - n), 1));
            return cycshft;
        }
    }
    //File 
    public class SecFunction
    {
        static ulong rtnulon;
        public static ulong Ch(ulong x,ulong y,ulong z)
        {
            rtnulon = (x & y) ^ (~x & z);
            return rtnulon;
        }
        public static ulong Maj(ulong x, ulong y, ulong z)
        {
            rtnulon = (x & y) ^ (x & z) ^ (y & z);
            return rtnulon;
        }

        //数据仅循环移位后相与
        public static ulong CyclicShift1(ulong x)
        {
            rtnulon = RingShift.CyclicShift(x, 28) ^ RingShift.CyclicShift(x, 34) ^ RingShift.CyclicShift(x, 39);
            return rtnulon;
        }
        public static ulong CyclicShift2(ulong x)
        {
            rtnulon = RingShift.CyclicShift(x, 14) ^ RingShift.CyclicShift(x, 18) ^ RingShift.CyclicShift(x, 41);
            return rtnulon;
        }

        //数据循环移位与移位后相与
        public static ulong CyclicShift3(ulong x)
        {
            rtnulon = RingShift.CyclicShift(x, 1) ^ RingShift.CyclicShift(x, 8) ^ RingShift.AbsoluteBit(x>>7, 7,1);
            return rtnulon;
        }
        public static ulong CyclicShift4(ulong x)
        {
            rtnulon = RingShift.CyclicShift(x, 19) ^ RingShift.CyclicShift(x, 61) ^ RingShift.AbsoluteBit(x >> 6, 6, 1);
            return rtnulon;
        }
    }
    //Tested
    public class Divide
    {
        //将文件等分为64bit，存于rtnlon数组
        //Tested
        public static ulong[] divText(string address)
        {
            if(!File.Exists(address))
            {
                Console.WriteLine("文件不存在");
                ulong[] rtnlons = new ulong[0xfffff];
                return rtnlons;
            }
            BinaryReader binred = new BinaryReader(File.Open(@address, FileMode.Open));
            //最大可以给出64M的文件的Hash值
            ulong[] rtnlon = new ulong[0xfffff];
            byte[] rtnlon2 = new byte[8];
            int i = 0;
            int j = 0;
            while (true)
            {
                if (binred.PeekChar() > -1)
                {
                    if (binred.BaseStream.Position + 8 > binred.BaseStream.Length)
                    {
                        while (binred.PeekChar() > -1)
                        {
                            rtnlon2[j] = binred.ReadByte();
                            j++;
                        }
                        rtnlon2[j] = 0x10;
                        j++;
                        while(j<8)
                        {
                            rtnlon2[j]=0;
                            j++;
                        }
                        rtnlon[i] = Merge.BytToUlon(rtnlon2, 0);
                        i++;
                    }
                    else
                    {
                        rtnlon[i] = binred.ReadUInt64();
                        i++;
                    }
                }
                else
                {
                    break;
                }
            }
            return rtnlon;
        }
    }
    //Tested
    public class FindW
    {
        public static ulong[] SeekW(ulong[] M)
        {
            ulong []W=new ulong[80];
            int i = 0;
            while (i < 16)
            {
                W[i] = M[i];
                i++;
            }
            while (i < 80)
            {
                W[i] = SecFunction.CyclicShift4(W[i - 2]) + W[i - 7] + SecFunction.CyclicShift3(W[i - 15]) + W[i - 16];
                 i++;
            }
            return W;
        }
    }
    //Tested
    public class EightRounds
    {
        //static ulong [,]HEgt=new ulong[81,8];
        public static ulong[] Rounds(ulong[] W)
        {
            ulong T1, T2;
            ulong[] H = new ulong[8];
            ulong a = BoxTransfer.H[0];
            ulong b = BoxTransfer.H[1];
            ulong c = BoxTransfer.H[2];
            ulong d = BoxTransfer.H[3];
            ulong e = BoxTransfer.H[4];
            ulong f = BoxTransfer.H[5];
            ulong g = BoxTransfer.H[6];
            ulong h = BoxTransfer.H[7];
            //表示轮数
            int t = 0;
            
            while (t < 80)
                {
                    T1 = h + SecFunction.CyclicShift2(e) + SecFunction.Ch(e, f, g) + BoxTransfer.K[t] + W[t];
                    T2 = SecFunction.CyclicShift1(a) + SecFunction.Maj(a, b, c);
                    h = g;
                    g = f;
                    f = e;
                    e = d + T1;
                    d = c;
                    c = b;
                    b = a;
                    a = T1 + T2;
                    t++;
                }
            H[0] = BoxTransfer.H2[0] + a;
            H[1] = BoxTransfer.H2[1] + b;
            H[2] = BoxTransfer.H2[2] + c;
            H[3] = BoxTransfer.H2[3] + d;
            H[4] = BoxTransfer.H2[4] + e;
            H[5] = BoxTransfer.H2[5] + f;
            H[6] = BoxTransfer.H2[6] + g;
            H[7] = BoxTransfer.H2[7] + h;
            return H;
        }
    }
    public class Call
    {
        //将数据分为1024bit分组并填充
        public static ulong[] BitToGrpAndGRes(string address)
        {
            //string str = "";
            ulong[] rtnres = new ulong[8];
            ulong[] M = new ulong[16];
            ulong[] W = new ulong[80];
            ulong[] rtnlon = Divide.divText(address);
            ulong i = 0;
            ulong j, k, l;
            while (rtnlon[i] != 0)
            {
                i++;
            }
            l = i;
            j = i % 16;
            k = j;
            if (0 < j && j < 13)
            {
                while (j < 15)
                {
                    rtnlon[i++] = 0;
                    j++;
                }
                rtnlon[15] = i * 64;
                //存储最终长度
                i = i + 16 - j;
            }
            else
            {
                while (j < 16)
                {
                    rtnlon[i++] = 0;
                    j++;
                }
                if (k == 0)
                {
                    rtnlon[--i] = l * 64;
                    i = l + 16;
                }
                else
                {
                    for (j = 0; j < 15; j++)
                    {
                        rtnlon[i++] = 0;

                    }
                    rtnlon[i++] = l * 64;
                }
            }
            //数组的总的长度，为16的整数倍
            k=i;
            l=k/16;
            i=0;
            j = 0;

            //开始进行调用整个函数
            while(i<l)
            {
                while (j < 16)
                {
                    M[j] = rtnlon[i * 16 + j];
                    j++;
                }
                i++;
                W = FindW.SeekW(M);
                BoxTransfer.H = EightRounds.Rounds(W);
                rtnres = BoxTransfer.H;                
            }
            Console.WriteLine("");

            //str = Merge.MergeResult(rtnres);
            return rtnres;
        }
    }

    public class Test
    {
        public static void Main()
        {
            int i;
            Console.WriteLine("Please input the file's address:");
            string address = Console.ReadLine();
            ulong[] result = Call.BitToGrpAndGRes(address);
            for (i = 0; i < 8; i++)
            {
                result[i] = result[i] + BoxTransfer.H2[i];
            }
            Console.WriteLine("The result is:");
            for (i = 0; i < 8; i++)
            {
                Console.WriteLine("{0:x}",result[i]);
            }
            Console.ReadLine();
        }
    }
}
```
