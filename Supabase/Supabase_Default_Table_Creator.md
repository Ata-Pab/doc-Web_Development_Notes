## Definitions

- ```REFERENCES auth.users``` is linked to the id in auth.users (Supabase's authentication system)
- ```ON DELETE CASCADE``` ensures that when a user is deleted, their profile entry is also deleted.
- ```CONSTRAINT``` is used to specify rules for the data in a table. Constraints are commonly used in SQL:
    - ```NOT NULL``` - Ensures that a column cannot have a NULL value
    - ```UNIQUE``` - Ensures that all values in a column are different
    - ```PRIMARY KEY``` - A combination of a NOT NULL and UNIQUE. Uniquely identifies each row in a table
    - ```FOREIGN KEY``` - Prevents actions that would destroy links between tables
    - ```CHECK``` - Ensures that the values in a column satisfies a specific condition
    - ```DEFAULT``` - Sets a default value for a column if no value is specified
    - ```CREATE INDEX``` - Used to create and retrieve data from the database very quickly
- ```CHAR_LENGTH``` returns the number of characters in an expression. [SQL Functions](https://docs.intersystems.com/irislatest/csp/docbook/DocBook.UI.Page.cls?KEY=RSQL_FUNCTIONS)
- ```ROW LEVEL SECURITY``` controls who can access or modify rows in the profiles table. By **default**, RLS **blocks all access** unless specific policies are set.
- ```USING``` (Row-Level Security for **SELECT**, **UPDATE**, **DELETE**): Determines which rows a user can interact with (read/update/delete). The condition inside USING is checked **before** allowing **access**.
- ```WITH CHECK``` (Row-Level Security for **INSERT**, **UPDATE**): Determines if a row satisfies conditions before being inserted/updated. The condition inside WITH CHECK is evaluated **after** an **INSERT** or **UPDATE** operation. If the row **fails** the WITH CHECK condition, the operation is **rejected**.
- ```search_path``` defines which database schemas PostgreSQL looks into when resolving object names. ```search_path = ''``` means no implicit schema search, ensuring that **all objects** must be explicitly referenced. The function ```public.handle_new_user()``` below explicitly refers to the public schema. So, even if another schema is active, it always inserts into ```public.profiles```, avoiding ambiguity.

```USING``` → Controls which rows a user can interact with (for SELECT, UPDATE, DELETE).

```WITH CHECK``` → Controls what values a user can insert/update (for INSERT, UPDATE).
### public.profiles
```sql
-- Create a table for public profiles
CREATE TABLE profiles (
  id UUID REFERENCES auth.users ON DELETE CASCADE NOT NULL PRIMARY KEY, 
  updated_at TIMESTAMP WITH TIME ZONE,
  username TEXT UNIQUE,
  full_name TEXT,
  avatar_url TEXT,
  website TEXT,

  CONSTRAINT username_length CHECK (CHAR_LENGTH(username) >= 3)
);

-- Set up Row Level Security (RLS)
-- See https://supabase.com/docs/guides/auth/row-level-security for more details.
ALTER TABLE profiles
  ENABLE ROW LEVEL SECURITY;

-- Anyone can view profiles but not modify them.
CREATE POLICY "Public profiles are viewable by everyone." ON profiles
  FOR SELECT USING (true);

-- A user can only insert their own profile, meaning: The id being inserted must match the currently authenticated user’s ID (auth.uid()).
CREATE POLICY "Users can insert their own profile." ON profiles
  FOR INSERT WITH CHECK ((SELECT auth.uid()) = id);

-- A user can update their own profile (where id = auth.uid()). They cannot update other users' profiles
CREATE POLICY "Users can update own profile." ON profiles
  FOR UPDATE USING ((SELECT auth.uid()) = id);

-- This trigger automatically creates a profile entry when a new user signs up via Supabase Auth.
-- See https://supabase.com/docs/guides/auth/managing-user-data#using-triggers for more details.
CREATE FUNCTION public.handle_new_user()
RETURNS trigger
SET search_path = ''
AS $$
BEGIN
  INSERT INTO public.profiles (id, full_name, avatar_url)
  VALUES (new.id, new.raw_user_meta_data->>'full_name', new.raw_user_meta_data->>'avatar_url');
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY definer;

-- Attach the Function as a Trigger. It automatically runs handle_new_user() whenever a new user is added to auth.users. It ensures that every new user gets a profile without needing to manually insert data
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();

-- Creates a storage bucket named avatars for storing user profile pictures.
INSERT INTO storage.buckets (id, name)
  VALUES ('avatars', 'avatars');

-- Set up access controls for storage.
-- See https://supabase.com/docs/guides/storage#policy-examples for more details.
CREATE POLICY "Avatar images are publicly accessible." ON storage.objects
  FOR SELECT USING (bucket_id = 'avatars');

-- Allows anyone to upload an image to the avatars bucket, they cannot overwrite or delete other users’ images
CREATE POLICY "Anyone can upload an avatar." ON storage.objects
  FOR INSERT WITH CHECK (bucket_id = 'avatars');

-- A user can update their own Avatar image (where id = auth.uid()). They cannot update other users' avatar images
-- CREATE POLICY "Users can update their own avatars." ON storage.objects
--    FOR UPDATE USING (metadata->>'owner' = auth.uid());  -- make sure you set metadata (owner: user.id) when uploading files!
```
