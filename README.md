# ClosetAI
AI stylist that creates outfits from your wardrobe
## Problem
People don’t know what to wear from their own wardrobe.

## Solution
ClosetAI analyzes your clothes and generates outfit combinations.

## MVP Features
- Upload clothes
- Categorize items
- Generate outfits

## Tech Stack
- Frontend:
- Backend:
- Al:
User
 └── Wardrobe
      ├── Tops
      ├── Bottoms
      ├── Shoes
      └── Accessories
closet-ai/
 ├── app/
 │    ├── page.tsx
 │    ├── wardrobe/
 │    └── api/
 ├── components/
 ├── lib/
 └── utils/
npx create-next-app@latest closet-ai
cd closet-ai
npm run dev
npm install @supabase/supabase-js
lib/supabase.ts
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
NEXT_PUBLIC_SUPABASE_URL=твоя_ссылка
NEXT_PUBLIC_SUPABASE_ANON_KEY=твой_ключ
create table items (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid,
  image_url text,
  category text,
  color text,
  created_at timestamp default now()
);
create table items (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid,
  image_url text,
  category text,
  color text,
  created_at timestamp default now()
);
'use client'

import { useState } from 'react'
import { supabase } from '@/lib/supabase'

export default function UploadPage() {
  const [file, setFile] = useState<File | null>(null)
  const [category, setCategory] = useState('')
  const [loading, setLoading] = useState(false)

  const handleUpload = async () => {
    if (!file) return alert('Выберите файл')

    setLoading(true)

    const fileName = `${Date.now()}-${file.name}`

    // 1. Загружаем файл в Storage
    const { error: uploadError } = await supabase.storage
      .from('clothes')
      .upload(fileName, file)

    if (uploadError) {
      alert(uploadError.message)
      setLoading(false)
      return
    }

    // 2. Получаем публичную ссылку
    const { data } = supabase.storage
      .from('clothes')
      .getPublicUrl(fileName)

    const imageUrl = data.publicUrl

    // 3. Сохраняем в таблицу
    const { error: dbError } = await supabase
      .from('items')
      .insert([
        {
          image_url: imageUrl,
          category: category,
        },
      ])

    if (dbError) {
      alert(dbError.message)
    } else {
      alert('Успешно загружено!')
      setFile(null)
      setCategory('')
    }

    setLoading(false)
  }

  return (
    <div className="min-h-screen flex flex-col items-center justify-center gap-4 p-6">
      <h1 className="text-2xl font-bold">Загрузка одежды 👕</h1>

      <input
        type="file"
        onChange={(e) => setFile(e.target.files?.[0] || null)}
      />

      <select
        value={category}
        onChange={(e) => setCategory(e.target.value)}
        className="border p-2 rounded"
      >
        <option value="">Выберите категорию</option>
        <option value="top">Верх</option>
        <option value="bottom">Низ</option>
        <option value="shoes">Обувь</option>
        <option value="accessories">Аксессуары</option>
      </select>

      <button
        onClick={handleUpload}
        disabled={loading}
        className="bg-black text-white px-4 py-2 rounded"
      >
        {loading ? 'Загрузка...' : 'Загрузить'}
      </button>
    </div>
  )
}
http://localhost:3000/upload
app/wardrobe/page.tsx
'use client'

import { useEffect, useState } from 'react'
import { supabase } from '@/lib/supabase'

type Item = {
  id: string
  image_url: string
  category: string
}

export default function WardrobePage() {
  const [items, setItems] = useState<Item[]>([])
  const [outfit, setOutfit] = useState<Item[]>([])

  useEffect(() => {
    fetchItems()
  }, [])

  const fetchItems = async () => {
    const { data } = await supabase.from('items').select('*')
    if (data) setItems(data)
  }

  const generateOutfit = () => {
    const tops = items.filter(i => i.category === 'top')
    const bottoms = items.filter(i => i.category === 'bottom')
    const shoes = items.filter(i => i.category === 'shoes')

    const random = (arr: Item[]) =>
      arr[Math.floor(Math.random() * arr.length)]

    const newOutfit = [
      tops.length ? random(tops) : null,
      bottoms.length ? random(bottoms) : null,
      shoes.length ? random(shoes) : null,
    ].filter(Boolean) as Item[]

    setOutfit(newOutfit)
  }

  return (
    <div className="min-h-screen p-6">
      <h1 className="text-3xl font-bold mb-6">Мой гардероб 👕</h1>

      <button
        onClick={generateOutfit}
        className="mb-6 bg-black text-white px-4 py-2 rounded"
      >
        Сгенерировать образ ✨
      </button>

      {outfit.length > 0 && (
        <div className="mb-8">
          <h2 className="text-xl font-semibold mb-4">Текущий образ:</h2>
          <div className="flex gap-4">
            {outfit.map(item => (
              <img
                key={item.id}
                src={item.image_url}
                className="w-40 h-40 object-cover rounded"
              />
            ))}
          </div>
        </div>
      )}

      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        {items.map(item => (
          <div key={item.id}>
            <img
              src={item.image_url}
              className="w-full h-40 object-cover rounded"
            />
            <p className="text-center mt-2">{item.category}</p>
          </div>
        ))}
      </div>
    </div>
  )
}
http://localhost:3000/wardrobe
alter table items
add column if not exists user_id uuid;
app/login/page.tsx
'use client'

import { useState } from 'react'
import { supabase } from '@/lib/supabase'
import { useRouter } from 'next/navigation'

export default function LoginPage() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const router = useRouter()

  const handleLogin = async () => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    })

    if (error) {
      alert(error.message)
    } else {
      router.push('/wardrobe')
    }
  }

  const handleRegister = async () => {
    const { error } = await supabase.auth.signUp({
      email,
      password,
    })

    if (error) {
      alert(error.message)
    } else {
      alert('Проверь почту для подтверждения!')
    }
  }

  return (
    <div className="min-h-screen flex flex-col items-center justify-center gap-4">
      <h1 className="text-2xl font-bold">Вход в ClosetAI 👗</h1>

      <input
        type="email"
        placeholder="Email"
        className="border p-2 rounded"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />

      <input
        type="password"
        placeholder="Пароль"
        className="border p-2 rounded"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />

      <button
        onClick={handleLogin}
        className="bg-black text-white px-4 py-2 rounded"
      >
        Войти
      </button>

      <button
        onClick={handleRegister}
        className="border px-4 py-2 rounded"
      >
        Зарегистрироваться
      </button>
    </div>
  )
}
const {
  data: { user },
} = await supabase.auth.getUser()

if (!user) return alert('Нужно войти в аккаунт')
await supabase.from('items').insert([
  {
    image_url: imageUrl,
    category: category,
    user_id: user.id,
  },
])
const fetchItems = async () => {
  const {
    data: { user },
  } = await supabase.auth.getUser()

  if (!user) return

  const { data } = await supabase
    .from('items')
    .select('*')
    .eq('user_id', user.id)

  if (data) setItems(data)
}
-- Разрешить читать только свои вещи
create policy "Users can view own items"
on items
for select
using (auth.uid() = user_id);

-- Разрешить добавлять только от своего имени
create policy "Users can insert own items"
on items
for insert
with check (auth.uid() = user_id);

-- Разрешить удалять только свои вещи
create policy "Users can delete own items"
on items
for delete
using (auth.uid() = user_id);

-- Разрешить обновлять только свои вещи
create policy "Users can update own items"
on items
for update
using (auth.uid() = user_id);
auth.uid() === user_id
.eq('user_id', user.id)
const { data } = await supabase
  .from('items')
  .select('*')
