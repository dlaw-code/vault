# Video walkthrough script

**Total speaking time: about 2:40 to 2:50** (417 spoken words at 150 to 160 words per minute). That leaves a little buffer under the 3:00 limit, so don't add much.

Have `docs/architecture.excalidraw` open and zoomed to fit. Lines in *[brackets]* tell you where to point and are not spoken.

---

## 1. Introduction (0:00 – 0:20, about 20 s)

*[Show the whole diagram.]*

> Hi, I'm ___. This is my vault program for the Turbin3 prerequisite. It's an Anchor program on Solana devnet that gives every wallet its own SOL vault, and withdrawing also registers you with Turbin3's on-chain registration program.

## 2. The accounts and PDAs (0:20 – 0:55, about 35 s)

*[Point at the **User wallet**, then follow the green dashed arrow down to **vault_state**.]*

> Everything starts with the user's wallet. It signs every instruction and pays the fees and rent.
>
> The program uses two PDAs. The first is `vault_state`, derived from the word "state" plus the user's public key. My program owns it, and it stores just two numbers: the bumps for itself and for the vault.

*[Move across to the **vault PDA** on the right.]*

> The second is the vault itself, derived from "vault" plus the `vault_state` address. It's a plain System account that only holds SOL.
>
> So the chain goes from user to `vault_state` to vault. There's no owner field. Ownership comes from the seeds, so another wallet derives completely different addresses.

## 3. initialize and deposit (0:55 – 1:20, about 25 s)

*[Point at **1. initialize()** and its two arrows.]*

> `initialize` creates `vault_state` and saves both bumps, so later instructions don't have to recompute them. It only *derives* the vault's address. The vault account doesn't exist until SOL is sent to it.

*[Point at **2. deposit()** and its arrow into the **System Program**.]*

> `deposit` is a simple transfer through the System Program, from the user into the vault. The user signs it like any normal payment.

## 4. withdraw and the registration CPI (1:20 – 2:15, about 55 s)

*[Point at **3. withdraw()** and its grey arrow to the System Program.]*

> `withdraw` is the interesting one. SOL now moves *out* of the vault, but the vault is a PDA with no private key. So my program signs for it using the vault's seeds and stored bump. Only this program can produce that signature, which keeps the funds safe.

*[Trace the thick purple arrow to the **registration program**, then to the **ApplicationAccount**.]*

> Then, in the same instruction, my program makes a cross-program invocation, a CPI, into Turbin3's registration program. It calls `initialize` with my GitHub username and passes three accounts: my wallet as the signer, the application account, and the System Program.
>
> My wallet's signature carries through the CPI, so the registration program creates an `ApplicationAccount` at "prereqs" plus my public key, funded by my wallet. Anchor checks I passed exactly that address. The account records my wallet and GitHub handle on-chain, and since it can only be created once, each wallet registers just once.

## 5. close (2:15 – 2:30, about 15 s)

*[Point at **4. close()** and its two arrows.]*

> Finally, `close` sends every lamport in the vault back to the user, with the program signing for the vault again. Then Anchor closes `vault_state` and refunds its rent to the user. Both accounts are gone.

## 6. Conclusion (2:30 – 2:40, about 10 s)

*[Point at the **Key ideas** at the bottom.]*

> So three ideas hold this together. Seeds define ownership, the program signs for the accounts it controls, and CPIs let one instruction move SOL and register me on another program at the same time. Thanks for watching.

---

## Facts checked against the source

Every claim in the script matches the code:

| Claim | Where it lives |
|---|---|
| `vault_state` seeds `["state", user]`, created with `init`, user pays | `initialize.rs` |
| Stores `vault_bump` and `state_bump` | `initialize.rs`, `state.rs` |
| vault seeds `["vault", vault_state]`, a `SystemAccount` with no `init` | `initialize.rs` |
| deposit transfers user → vault with a plain `CpiContext::new` (user signs) | `deposit.rs` |
| withdraw and close sign with `new_with_signer` and `["vault", vault_state, vault_bump]` | `withdraw.rs`, `close.rs` |
| close sends `vault.lamports()` (everything) and then `close = user` | `close.rs` |
| Application account seeds `["prereqs", user]`, checked with `seeds::program` | `withdraw.rs` |
| Registration `initialize(github)` takes user (signer), account, system_program | `idls/registration.json` |

**Before recording:** the registration CPI in section 4 must actually be in `withdraw.rs`, and your withdraw test must pass on devnet. Until then, that section describes code that doesn't exist yet.

## Recording tips

- **Rehearse with a timer twice.** If you run over, shorten section 4's last paragraph first. Don't cut the PDA signing explanation.
- **Speak in your own words.** Use the script as a guide, not something to read out. Evaluators can hear reading, and they're grading whether you understand the design.
- **Point with the cursor**, or select each box in Excalidraw as you mention it. Zooming in slightly on withdraw and the CPI arrow helps during section 4.
- **Recording tools:** OBS Studio or Loom work well. Windows' Snipping Tool also records the screen. Record at 1080p.
- **Captions:** after uploading, open YouTube Studio → Subtitles, let auto-captions generate, then fix terms like *PDA*, *Anchor*, *lamports* and *CPI*.
- **Visibility:** set the video to **Public** or **Unlisted**, then test the link in a private or incognito window.
