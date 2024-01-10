# My Exprience about Node (Nuxt 3 w Prisma)
## Installation And Build
- Init new App Nuxt 3
- Install Prisma(Dev) and @prisma/client
- Run `npx prisma init`
- Configure or Add model for database engine in file `schema.prisma`
- After configure and add the model, Run `npx prisma migrate dev --name init` to migrate database.
- Try to Run `npm run dev` or `yarn dev`
- (Skip if not error) If you got the error `DATABASE_URL` Bla..bla..blaaa
- Before Building App run `npx prisma generate`
- And last `npm run build` or `yarn build`
- Try to run `node .output/server/index.mjs`
- Done!
