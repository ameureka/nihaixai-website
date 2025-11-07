# Phase 3: 用户交互 - 设计文档

## 设计概述

**设计目标：** 构建完整的用户交互系统，提升用户参与度和社区活跃度
**技术栈：** NextAuth.js + Zustand + SWR + Prisma + next-i18next
**设计原则：** 简单易用、安全可靠、性能优秀、多语言友好
**数据模型集成：** 基于Phase 1的统一Content模型  

---

## 系统架构

### 整体架构图

```
用户浏览器
    ↓
Next.js App
    ↓
├── NextAuth.js (认证)
│   └── Google OAuth Provider
├── Zustand (状态管理)
├── SWR (数据获取)
├── API Routes
│   ├── /api/auth/* (认证)
│   ├── /api/comments/* (评论)
│   ├── /api/likes/* (点赞)
│   └── /api/shares/* (分享)
└── Prisma ORM
    └── Neon PostgreSQL
```

---

## 用户认证设计

### NextAuth.js 配置

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth, { NextAuthOptions } from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import { PrismaAdapter } from '@next-auth/prisma-adapter';
import { prisma } from '@/lib/db';

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, user }) {
      if (session.user) {
        session.user.id = user.id;
        session.user.role = user.role;
      }
      return session;
    },
  },
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error',
  },
};

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

### 数据库 Schema 扩展

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  role          Role      @default(USER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  accounts      Account[]
  sessions      Session[]
  comments      Comment[]
  likes         Like[]
  
  @@map("users")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([provider, providerAccountId])
  @@map("accounts")
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@map("sessions")
}

model Comment {
  id        String   @id @default(cuid())
  content   String   @db.Text
  status    CommentStatus @default(PENDING)
  contentId String   // 关联到统一Content模型（支持ARTICLE、TIANJI_PAGE等所有类型）
  userId    String
  parentId  String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  content   Content   @relation(fields: [contentId], references: [id], onDelete: Cascade)
  parent    Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies   Comment[] @relation("CommentReplies")
  likes     Like[]

  @@map("comments")
  @@index([contentId])
  @@index([userId])
  @@index([status])
}

model Like {
  id        String   @id @default(cuid())
  userId    String
  commentId String?  // 点赞评论（可选）
  contentId String?  // 点赞内容（可选，支持所有ContentType）
  createdAt DateTime @default(now())

  user    User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  comment Comment? @relation(fields: [commentId], references: [id], onDelete: Cascade)
  content Content? @relation(fields: [contentId], references: [id], onDelete: Cascade)

  @@unique([userId, commentId])
  @@unique([userId, contentId])
  @@map("likes")
  @@index([userId])
}

enum Role {
  USER
  ADMIN
}

enum CommentStatus {
  PENDING
  APPROVED
  REJECTED
}
```

### 认证组件

```typescript
// components/auth/SignInButton.tsx
'use client';

import { signIn, signOut, useSession } from 'next-auth/react';
import Image from 'next/image';

export function SignInButton() {
  const { data: session, status } = useSession();
  
  if (status === 'loading') {
    return <div className="w-8 h-8 bg-gray-200 rounded-full animate-pulse" />;
  }
  
  if (session) {
    return (
      <div className="relative group">
        <button className="flex items-center space-x-2">
          <Image
            src={session.user.image || '/default-avatar.png'}
            alt={session.user.name || 'User'}
            width={32}
            height={32}
            className="rounded-full"
          />
          <span className="hidden md:block text-sm font-medium">
            {session.user.name}
          </span>
        </button>
        
        <div className="absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-lg py-2 hidden group-hover:block">
          <a href="/profile" className="block px-4 py-2 text-sm hover:bg-gray-100">
            个人中心
          </a>
          <button
            onClick={() => signOut()}
            className="block w-full text-left px-4 py-2 text-sm hover:bg-gray-100"
          >
            退出登录
          </button>
        </div>
      </div>
    );
  }
  
  return (
    <button
      onClick={() => signIn('google')}
      className="flex items-center space-x-2 px-4 py-2 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors"
    >
      <svg className="w-5 h-5" viewBox="0 0 24 24">
        {/* Google icon */}
      </svg>
      <span className="text-sm font-medium">使用 Google 登录</span>
    </button>
  );
}
```

---

## 评论系统设计

### 评论 API

```typescript
// app/api/comments/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import { prisma } from '@/lib/db';
import { z } from 'zod';

const commentSchema = z.object({
  content: z.string().min(1).max(1000),
  contentId: z.string(),  // 支持所有Content类型（ARTICLE、TIANJI_PAGE、DIJI_PAGE等）
  parentId: z.string().optional(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session?.user) {
      return NextResponse.json(
        { error: '请先登录' },
        { status: 401 }
      );
    }
    
    const body = await request.json();
    const { content, contentId, parentId } = commentSchema.parse(body);

    const comment = await prisma.comment.create({
      data: {
        content,
        contentId,
        parentId,
        userId: session.user.id,
        status: 'PENDING',
      },
      include: {
        user: {
          select: {
            name: true,
            image: true,
          },
        },
        content: {
          select: {
            type: true,
            slug: true,
            title: true,
          },
        },
      },
    });
    
    return NextResponse.json(comment);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: '输入数据无效' },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: '创建评论失败' },
      { status: 500 }
    );
  }
}

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const contentId = searchParams.get('contentId');

    if (!contentId) {
      return NextResponse.json(
        { error: '缺少 contentId 参数' },
        { status: 400 }
      );
    }

    const comments = await prisma.comment.findMany({
      where: {
        contentId,
        status: 'APPROVED',
        parentId: null,
      },
      include: {
        user: {
          select: {
            name: true,
            image: true,
          },
        },
        replies: {
          include: {
            user: {
              select: {
                name: true,
                image: true,
              },
            },
            likes: true,
          },
        },
        likes: true,
      },
      orderBy: {
        createdAt: 'desc',
      },
    });
    
    return NextResponse.json(comments);
  } catch (error) {
    return NextResponse.json(
      { error: '获取评论失败' },
      { status: 500 }
    );
  }
}
```

### 评论组件

```typescript
// components/comments/CommentSection.tsx
'use client';

import { useState } from 'react';
import { useSession } from 'next-auth/react';
import useSWR from 'swr';
import { CommentForm } from './CommentForm';
import { CommentList } from './CommentList';

interface CommentSectionProps {
  contentId: string;  // 支持所有Content类型（ARTICLE、TIANJI_PAGE等）
}

export function CommentSection({ contentId }: CommentSectionProps) {
  const { data: session } = useSession();
  const { data: comments, mutate } = useSWR(
    `/api/comments?contentId=${contentId}`,
    fetcher
  );

  const handleCommentSubmit = async (content: string) => {
    const response = await fetch('/api/comments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ content, contentId }),
    });

    if (response.ok) {
      mutate();
    }
  };
  
  return (
    <div className="mt-12">
      <h2 className="text-2xl font-bold mb-6">评论</h2>
      
      {session ? (
        <CommentForm onSubmit={handleCommentSubmit} />
      ) : (
        <div className="bg-gray-50 rounded-lg p-6 text-center">
          <p className="text-gray-600 mb-4">登录后参与讨论</p>
          <SignInButton />
        </div>
      )}
      
      {comments && comments.length > 0 ? (
        <CommentList comments={comments} contentId={contentId} />
      ) : (
        <p className="text-gray-500 text-center py-8">暂无评论，来发表第一条评论吧！</p>
      )}
    </div>
  );
}
```

```typescript
// components/comments/CommentForm.tsx
'use client';

import { useState } from 'react';

interface CommentFormProps {
  onSubmit: (content: string) => Promise<void>;
  placeholder?: string;
  parentId?: string;
}

export function CommentForm({ 
  onSubmit, 
  placeholder = '写下你的评论...',
  parentId 
}: CommentFormProps) {
  const [content, setContent] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!content.trim()) return;
    
    setIsSubmitting(true);
    try {
      await onSubmit(content);
      setContent('');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="mb-8">
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder={placeholder}
        className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-transparent resize-none"
        rows={4}
        maxLength={1000}
      />
      
      <div className="flex items-center justify-between mt-2">
        <span className="text-sm text-gray-500">
          {content.length}/1000
        </span>
        
        <button
          type="submit"
          disabled={!content.trim() || isSubmitting}
          className="px-6 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
        >
          {isSubmitting ? '提交中...' : '发表评论'}
        </button>
      </div>
    </form>
  );
}
```

```typescript
// components/comments/CommentItem.tsx
'use client';

import { useState } from 'react';
import { useSession } from 'next-auth/react';
import Image from 'next/image';
import { formatDistanceToNow } from 'date-fns';
import { zhCN } from 'date-fns/locale';

interface CommentItemProps {
  comment: Comment;
  onReply?: (content: string) => Promise<void>;
  onLike?: () => Promise<void>;
}

export function CommentItem({ comment, onReply, onLike }: CommentItemProps) {
  const { data: session } = useSession();
  const [showReplyForm, setShowReplyForm] = useState(false);
  const [isLiked, setIsLiked] = useState(
    comment.likes.some(like => like.userId === session?.user?.id)
  );
  
  const handleLike = async () => {
    if (!session) return;
    
    setIsLiked(!isLiked);
    await onLike?.();
  };
  
  return (
    <div className="flex space-x-4">
      <Image
        src={comment.user.image || '/default-avatar.png'}
        alt={comment.user.name}
        width={40}
        height={40}
        className="rounded-full"
      />
      
      <div className="flex-1">
        <div className="bg-gray-50 rounded-lg p-4">
          <div className="flex items-center justify-between mb-2">
            <span className="font-medium text-gray-900">
              {comment.user.name}
            </span>
            <span className="text-sm text-gray-500">
              {formatDistanceToNow(new Date(comment.createdAt), {
                addSuffix: true,
                locale: zhCN,
              })}
            </span>
          </div>
          
          <p className="text-gray-700 whitespace-pre-wrap">
            {comment.content}
          </p>
        </div>
        
        <div className="flex items-center space-x-4 mt-2">
          <button
            onClick={handleLike}
            className={cn(
              "flex items-center space-x-1 text-sm transition-colors",
              isLiked ? "text-primary-600" : "text-gray-500 hover:text-primary-600"
            )}
          >
            <svg className="w-4 h-4" fill={isLiked ? "currentColor" : "none"} stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M14 10h4.764a2 2 0 011.789 2.894l-3.5 7A2 2 0 0115.263 21h-4.017c-.163 0-.326-.02-.485-.06L7 20m7-10V5a2 2 0 00-2-2h-.095c-.5 0-.905.405-.905.905 0 .714-.211 1.412-.608 2.006L7 11v9m7-10h-2M7 20H5a2 2 0 01-2-2v-6a2 2 0 012-2h2.5" />
            </svg>
            <span>{comment.likes.length}</span>
          </button>
          
          {session && (
            <button
              onClick={() => setShowReplyForm(!showReplyForm)}
              className="text-sm text-gray-500 hover:text-primary-600 transition-colors"
            >
              回复
            </button>
          )}
        </div>
        
        {showReplyForm && onReply && (
          <div className="mt-4">
            <CommentForm
              onSubmit={async (content) => {
                await onReply(content);
                setShowReplyForm(false);
              }}
              placeholder={`回复 ${comment.user.name}...`}
            />
          </div>
        )}
        
        {comment.replies && comment.replies.length > 0 && (
          <div className="mt-4 space-y-4">
            {comment.replies.map((reply) => (
              <CommentItem key={reply.id} comment={reply} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 社交分享设计

### 分享组件

```typescript
// components/share/ShareButtons.tsx
'use client';

import { useState } from 'react';

interface ShareButtonsProps {
  url: string;
  title: string;
  description?: string;
}

export function ShareButtons({ url, title, description }: ShareButtonsProps) {
  const [showQRCode, setShowQRCode] = useState(false);
  const [copied, setCopied] = useState(false);
  
  const shareUrl = `${url}?utm_source=share&utm_medium=social`;
  
  const handleWeChatShare = () => {
    setShowQRCode(true);
  };
  
  const handleWeiboShare = () => {
    const weiboUrl = `https://service.weibo.com/share/share.php?url=${encodeURIComponent(shareUrl)}&title=${encodeURIComponent(title)}`;
    window.open(weiboUrl, '_blank', 'width=600,height=400');
  };
  
  const handleTwitterShare = () => {
    const twitterUrl = `https://twitter.com/intent/tweet?url=${encodeURIComponent(shareUrl)}&text=${encodeURIComponent(title)}`;
    window.open(twitterUrl, '_blank', 'width=600,height=400');
  };
  
  const handleCopyLink = async () => {
    try {
      await navigator.clipboard.writeText(shareUrl);
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
      
      // 记录分享事件
      await fetch('/api/analytics/share', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ url, platform: 'copy' }),
      });
    } catch (error) {
      console.error('复制失败:', error);
    }
  };
  
  return (
    <div className="flex items-center space-x-4">
      <span className="text-sm font-medium text-gray-700">分享到：</span>
      
      <button
        onClick={handleWeChatShare}
        className="p-2 text-green-600 hover:bg-green-50 rounded-lg transition-colors"
        title="分享到微信"
      >
        <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          {/* WeChat icon */}
        </svg>
      </button>
      
      <button
        onClick={handleWeiboShare}
        className="p-2 text-red-600 hover:bg-red-50 rounded-lg transition-colors"
        title="分享到微博"
      >
        <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          {/* Weibo icon */}
        </svg>
      </button>
      
      <button
        onClick={handleTwitterShare}
        className="p-2 text-blue-500 hover:bg-blue-50 rounded-lg transition-colors"
        title="分享到 Twitter"
      >
        <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          {/* Twitter icon */}
        </svg>
      </button>
      
      <button
        onClick={handleCopyLink}
        className="p-2 text-gray-600 hover:bg-gray-100 rounded-lg transition-colors"
        title="复制链接"
      >
        {copied ? (
          <svg className="w-5 h-5 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
          </svg>
        ) : (
          <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
          </svg>
        )}
      </button>
      
      {showQRCode && (
        <QRCodeModal
          url={shareUrl}
          onClose={() => setShowQRCode(false)}
        />
      )}
    </div>
  );
}
```

---

## 状态管理设计

### Zustand Store

```typescript
// lib/store/useUserStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface UserState {
  user: User | null;
  setUser: (user: User | null) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserState>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      clearUser: () => set({ user: null }),
    }),
    {
      name: 'user-storage',
    }
  )
);
```

### SWR 配置

```typescript
// lib/swr-config.ts
import { SWRConfig } from 'swr';

const fetcher = async (url: string) => {
  const response = await fetch(url);
  
  if (!response.ok) {
    const error = new Error('请求失败');
    error.info = await response.json();
    error.status = response.status;
    throw error;
  }
  
  return response.json();
};

export const swrConfig = {
  fetcher,
  revalidateOnFocus: false,
  revalidateOnReconnect: true,
  shouldRetryOnError: true,
  errorRetryCount: 3,
  errorRetryInterval: 5000,
};

// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <SWRConfig value={swrConfig}>
          {children}
        </SWRConfig>
      </body>
    </html>
  );
}
```

---

## 多语言用户交互设计

### 翻译资源配置

```json
// public/locales/zh-CN/user-interaction.json
{
  "auth": {
    "signIn": "登录",
    "signOut": "退出登录",
    "signInWith": "使用 {{provider}} 登录",
    "profile": "个人中心",
    "pleaseSignIn": "请先登录"
  },
  "comments": {
    "title": "评论",
    "placeholder": "写下你的评论...",
    "reply": "回复",
    "like": "点赞",
    "submit": "发表评论",
    "submitting": "提交中...",
    "noComments": "暂无评论，来发表第一条评论吧！",
    "loginToComment": "登录后参与讨论",
    "replyTo": "回复 {{name}}...",
    "pending": "等待审核",
    "approved": "已发布",
    "rejected": "已拒绝"
  },
  "share": {
    "title": "分享到：",
    "wechat": "分享到微信",
    "weibo": "分享到微博",
    "twitter": "分享到 Twitter",
    "copyLink": "复制链接",
    "linkCopied": "链接已复制"
  },
  "errors": {
    "loginRequired": "请先登录",
    "invalidInput": "输入数据无效",
    "commentFailed": "创建评论失败",
    "loadFailed": "加载失败",
    "networkError": "网络错误，请稍后重试"
  }
}

// public/locales/zh-TW/user-interaction.json
{
  "auth": {
    "signIn": "登入",
    "signOut": "登出",
    "signInWith": "使用 {{provider}} 登入",
    "profile": "個人中心",
    "pleaseSignIn": "請先登入"
  },
  "comments": {
    "title": "留言",
    "placeholder": "寫下你的留言...",
    "reply": "回覆",
    "like": "讚",
    "submit": "發表留言",
    "submitting": "提交中...",
    "noComments": "暫無留言，來發表第一條留言吧！",
    "loginToComment": "登入後參與討論",
    "replyTo": "回覆 {{name}}...",
    "pending": "等待審核",
    "approved": "已發佈",
    "rejected": "已拒絕"
  },
  "share": {
    "title": "分享到：",
    "wechat": "分享到微信",
    "weibo": "分享到微博",
    "twitter": "分享到 Twitter",
    "copyLink": "複製連結",
    "linkCopied": "連結已複製"
  },
  "errors": {
    "loginRequired": "請先登入",
    "invalidInput": "輸入數據無效",
    "commentFailed": "創建留言失敗",
    "loadFailed": "載入失敗",
    "networkError": "網路錯誤，請稍後重試"
  }
}

// public/locales/en/user-interaction.json
{
  "auth": {
    "signIn": "Sign In",
    "signOut": "Sign Out",
    "signInWith": "Sign in with {{provider}}",
    "profile": "Profile",
    "pleaseSignIn": "Please sign in first"
  },
  "comments": {
    "title": "Comments",
    "placeholder": "Write your comment...",
    "reply": "Reply",
    "like": "Like",
    "submit": "Post Comment",
    "submitting": "Submitting...",
    "noComments": "No comments yet. Be the first to comment!",
    "loginToComment": "Sign in to join the discussion",
    "replyTo": "Reply to {{name}}...",
    "pending": "Pending",
    "approved": "Approved",
    "rejected": "Rejected"
  },
  "share": {
    "title": "Share:",
    "wechat": "Share on WeChat",
    "weibo": "Share on Weibo",
    "twitter": "Share on Twitter",
    "copyLink": "Copy Link",
    "linkCopied": "Link Copied"
  },
  "errors": {
    "loginRequired": "Please sign in first",
    "invalidInput": "Invalid input",
    "commentFailed": "Failed to create comment",
    "loadFailed": "Loading failed",
    "networkError": "Network error, please try again later"
  }
}
```

### 多语言组件示例

```typescript
// components/auth/SignInButton.tsx (多语言版本)
'use client';

import { useTranslation } from 'next-i18next';
import { signIn, signOut, useSession } from 'next-auth/react';

export function SignInButton() {
  const { t } = useTranslation('user-interaction');
  const { data: session, status } = useSession();

  if (session) {
    return (
      <div className="relative group">
        <button className="flex items-center space-x-2">
          <Image src={session.user.image || '/default-avatar.png'} />
          <span>{session.user.name}</span>
        </button>

        <div className="dropdown-menu">
          <a href="/profile">{t('auth.profile')}</a>
          <button onClick={() => signOut()}>
            {t('auth.signOut')}
          </button>
        </div>
      </div>
    );
  }

  return (
    <button onClick={() => signIn('google')}>
      {t('auth.signInWith', { provider: 'Google' })}
    </button>
  );
}
```

```typescript
// components/comments/CommentSection.tsx (多语言版本)
'use client';

import { useTranslation } from 'next-i18next';

export function CommentSection({ contentId }: CommentSectionProps) {
  const { t } = useTranslation('user-interaction');
  const { data: session } = useSession();

  return (
    <div className="mt-12">
      <h2 className="text-2xl font-bold mb-6">{t('comments.title')}</h2>

      {session ? (
        <CommentForm
          onSubmit={handleCommentSubmit}
          placeholder={t('comments.placeholder')}
          submitText={t('comments.submit')}
          submittingText={t('comments.submitting')}
        />
      ) : (
        <div className="bg-gray-50 rounded-lg p-6 text-center">
          <p className="text-gray-600 mb-4">{t('comments.loginToComment')}</p>
          <SignInButton />
        </div>
      )}

      {comments && comments.length > 0 ? (
        <CommentList comments={comments} contentId={contentId} />
      ) : (
        <p className="text-gray-500 text-center py-8">
          {t('comments.noComments')}
        </p>
      )}
    </div>
  );
}
```

### 多语言分享文案生成

```typescript
// lib/share-utils.ts
import { TFunction } from 'next-i18next';

export function generateShareText(
  content: ContentWithTranslation,
  locale: string,
  t: TFunction
): string {
  const title = content.translation.title;
  const description = content.translation.description;

  // 根据locale生成不同的分享文案
  if (locale === 'zh-CN') {
    return `分享：${title} - ${description}`;
  } else if (locale === 'zh-TW') {
    return `分享：${title} - ${description}`;
  } else {
    return `Check this out: ${title} - ${description}`;
  }
}
```

---

**文档版本：** 1.1.0
**创建日期：** 2024-11-07
**最后更新：** 2025-01-07
**文档状态：** ✅ 已完成
**更新说明：**
- 更新数据模型以使用统一Content模型（contentId替代articleId）
- 添加多语言用户交互支持（基于next-i18next）
- 更新所有API和组件示例
