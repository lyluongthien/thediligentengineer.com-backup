---
title: "Manual Transaction Injection in KeystoneJS: A Workaround for Missing Transaction Support"
datePublished: Sun Jan 14 2024 18:53:19 GMT+0000 (Coordinated Universal Time)
cuid: clrdutlzk000208l8d2mjafhe
slug: manual-transaction-injection-in-keystonejs-a-workaround-for-missing-transaction-support
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705258445677/d0d3bb72-ba41-4c65-ac93-82aa2c41e2a9.png
tags: typescript

---


# Manual Transaction Injection in KeystoneJS: A Workaround for Missing Transaction Support

KeystoneJS's Lack of Native Transaction Support

-   KeystoneJS currently lacks built-in support for database transactions, posing challenges for reliable data integrity and error handling.
  See: https://github.com/keystonejs/keystone/discussions/7642
-   This limitation necessitates manual rollbacks, increasing the risk of inconsistencies and potential data loss.
-   Using Prisma's transactions directly bypasses Keystone's access control and hooks, hindering its core functionalities:
    > “This API executes the access control rules and hooks defined in your system. To bypass these, you can directly use the Prisma Client at `context.prisma`.”
    
    See: https://keystonejs.com/docs/context/db-items#database


## A Workaround: Manual Transaction Injection

To address this issue, here's a solution that demonstrates manual transaction injection in KeystoneJS:

### 1\. Defining the `PostgresTransaction` Interface:

```ts
export interface PostgresTransaction {
  name: string;
  commit: () => Promise<void>;
  rollback: () => Promise<void>;
}

```



### 2\. Creating a Transaction:

```ts
 

export const createPostgresTransaction = async (
  context: AuthenticatedContext | Context,
  name: string,
  timeout = 30_000,
): Promise<PostgresTransaction> => {
  const parentTransaction = (<AuthenticatedContext>context).parentTransaction;
  if (parentTransaction) {
    return {
      name: parentTransaction.name,
      commit: async () => {
        logInfo(`[${name}] Transaction commit is ignored because it is a parent transaction ${parentTransaction.name}`);
      },
      rollback: async (err) => {
        logInfo(`Transaction ${parentTransaction.name} rolled back in a child transaction ${name}`);
        await parentTransaction.rollback(err);
      },
    };
  }
  const prismaClient = context.prisma;
  let committed = false;
  let rolledBack = false;

  const timer = setTimeout(async () => {
    if (rolledBack || committed) return;
    await prismaClient.$queryRaw`ROLLBACK`;
    logInfo(`[${name}] Transaction timeout`);
  }, timeout);
  await prismaClient.$queryRaw`START TRANSACTION`;
  logInfo(`[${name}] Transaction started`);
  const tran = {
    name,
    /**
     * End transaction
     */
    commit: async () => {
      if (rolledBack) {
        logInfo(`[${name}] Transaction has been rolled back`);
        return;
      }
      await prismaClient.$queryRaw`COMMIT`;
      logInfo(`[${name}] Transaction committed`);
      committed = true;
      clearTimeout(timer);
    },
    rollback: async (error?: unknown) => {
      if (committed) {
        // this error should not happen, but if yes, you must debug it asap
        throw new Error(`[${name}] Transaction has been committed`);
      }
      await prismaClient.$queryRaw`ROLLBACK`;
      logInfo(`[${name}] Transaction rolled back due to:`, error);
      rolledBack = true;
      clearTimeout(timer);
    },
  };
  (<AuthenticatedContext>context).parentTransaction = tran;
  return tran;
};


```



## Key Features:

-   Nested Transaction Handling: Ensures correct commit and rollback behavior within nested transactions.
-   Timeout Mechanism: Prevents indefinite transaction locks with a 30-second timeout.
-   Logging: Provides informative logs for debugging and monitoring transaction activity.

## Example Usage:

```ts
async function functionA(context) {
  const transactionA = await createPostgresTransaction(context, "Transaction A");

  try {
    // Perform actions within transaction A

    // Call functionB, which starts its own transaction
    await functionB(context); 
    // because of nested transaction handling, transaction 
    // in functionB will be ignored and functionA will be the only transaction 
    // that wraps all the actions in functionB


    await transactionA.commit(); // Commit transaction A
  } catch (error) {
    await transactionA.rollback(); // Rollback transaction A if any error occurs
  }
}

async function functionB() {
  const transactionB = await createPostgresTransaction(context, "Transaction B");

  try {
    // Perform actions within transaction B

    await transactionB.commit(); // Commit transaction B
  } catch (error) {
    await transactionB.rollback(); // Rollback transaction B if any error occurs
    // Note: This will rollback any parent transactions as well if exist
    throw error; // Re-throw the error to allow functionA to handle it
  }
}


```



## Addressing Limitations and Considerations:

-   Manual Approach: This workaround involves manual management, increasing development overhead.
-   Potential Integration Issues: Ensure compatibility with other Keystone features and middleware.
-   Future Native Support: Advocate for native transaction support within KeystoneJS for a more streamlined approach.

## Conclusion:

While this workaround provides a viable solution, it highlights the need for native transaction support within KeystoneJS. Until then, consider this approach for enhanced data integrity and error handling in your KeystoneJS projects.