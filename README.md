Ultra Street Fighter II — reconstruction kit

**STREAMLINED: DROP YOUR FILES IN THE BUILDER AND GET THE BACKPORT!**

Version: rc2 (2026-10-17)

Backport by: Steve Gordon (https://x.com/strygo)



Ultra Street Fighter II: The Final Challengers is the 2017 Nintendo Switch edition of Super Street Fighter II Turbo, and it never had an arcade release. This backport rebuilds it as a CPS-2 game.

Akuma has his own select cell, and Evil Ryu and Violent Ken join the original sixteen fighters with their Ultra moves, colors and endings. Shin Akuma stays hidden, for twenty fighters in all.

Ultra's presentation comes with them: the Ultra title and intro, "The Final Challengers" lettering, a color select for the new fighters, and a choice of sixteen stages or Random once you have picked a fighter.

The Switch edition's sound effects play in every build, and on MiSTer its arranged music plays too. The ULTRA CONFIGURATION page in Test Mode switches the sound effects and, on MiSTer, the music between MODERN and CLASSIC.

The USA build is in English; the Japan build keeps the Japanese.

This is a release candidate.



Changes:

- Akuma on his own select cell, with Evil Ryu and Violent Ken: twenty fighters with the hidden Shin Akuma

- Ultra's classic colors for the new fighters, chosen on a color select screen

- Sixteen stages and Random to choose from after you pick a fighter

- The Ultra title and intro, and ULTRA on the VS, Continue and Game Over screens

- The Switch edition's sound effects, and its arranged music on MiSTer; MODERN or CLASSIC for each in ULTRA CONFIGURATION

- USA (English) and Japan builds

INPUTS
  - Canonical MAME-format ssf2xj and its parent/shared members (ssf2t).
    Complete, merged, split ZIP/7z and loose layouts are supported by hash.
    7z needs installed 7-Zip. Supply canonical QSound dl-1425.bin as well.
  - USA base USF2 NSP for USA output; Japanese base NSP for Japanese output.
    Both are required for --region all. Supported identities are in kit.json.
    Update NSPs are not required. Filenames are not identities.
  - Your own prod.keys. Keys are never bundled or downloaded.
  - Python 3.10+, git, CMake and a C/C++ compiler for pinned NSTool/libopus.
  - MiSTer music generation additionally needs NumPy and FFmpeg.
    It uses fixed-point Opus decoding and the qualified ADX conversion. Any
    differing final music bytes fail the published CPK hash check.

BUILD ALL OUTPUTS
  python3 apply.py --rompath ROM_DIRECTORY --nsp-us USA_BASE.nsp \
    --nsp-jp JAPAN_BASE.nsp --keys prod.keys --work scratch --out-dir output

  --region us|jp|all      Select regional output (default: all).
  --audio 4|8|all         Restrict sample capacity (default: all).
  --platform mame|hbmame|mister|all (default: all).
  --check                Verify arcade inputs and decrypt NSP resources only.

  Output must be fresh. Inputs are never modified. The build stages privately
  and publishes the completed tree only after exact ROM/MRA/music checks pass.
  Source tools are fetched from pinned revisions; no game data is downloaded.

WINDOWS TOOL BUILD
  Use Visual Studio 2019 or newer with Desktop development with C++, or
  MinGW-w64. If an older kit fails building libmbedtls with C2059/C5103 in
  Windows SDK oaidl.h/wtypes.h (_VARIANT_BOOL), install Windows SDK
  10.0.20348.0 or later through Visual Studio Installer > Modify > Individual
  components. Rerun the same apply.py command with a new --work directory
  (for example, --work scratch-sdk) so CMake selects the newly installed SDK.
  The updated source builder avoids forcing C11 mode on MSVC for NSTool's
  legacy C dependencies. This failure occurs before ROM reconstruction.

OUTPUTS
  mame/hsf2.zip               USA, stock MAME (4 MiB MODERN audio).
  mame/hsf2j.zip              Japan, stock MAME (4 MiB MODERN audio).
  mame/RUN-MAME.txt           Launch commands for stock MAME.
  hbmame/<set>.zip            USA/Japan 8 MiB CLASSIC/MODERN game archives.
  definitions/*.cpp.inc       Exact ROM and GAME registrations for all four.
  mister/games/mame/          Complete native archives (firmware included).
  mister/games/CPS+/          The generated usf2_arrange.zip / CPK.
  mister/_Arcade/             Two 8 MiB CPS+ MRAs and the CPS2+ core.
  verification.json          Exact selected profile and assembled-image hashes.

  Copy the contents of mister/ to the MiSTer card. Both MRAs use the included
  CPS2+ core and require 128 MiB SDRAM. Demo sounds, MODERN sound effects and
  MODERN music are enabled by default. In Test Mode, ULTRA CONFIGURATION has
  SOUND EFFECTS and MUSIC, each using MODERN / CLASSIC. MUSIC appears only
  when the MRA enables it. The unmodified ROM hides it on emulators/hardware.
  Existing saved settings take precedence. Clear that launcher's NVRAM to
  reload the supplied MRA EEPROM defaults. MAME and HBMAME retain sequenced
  QSound music. The 4 MiB version always uses MODERN effects, with reduced
  rates for less-used speech to fit its smaller sample space.

MAME
  The MAME ZIPs run in unmodified MAME using its existing Hyper Street
  Fighter II loader. Keep them in a separate folder from original HSF2 ROMs.
  From your MAME folder, launch USA or Japan:
    mame hsf2 -rompath "FULL_PATH_TO_OUTPUT/mame" -nvram_directory nvram-usf2
    mame hsf2j -rompath "FULL_PATH_TO_OUTPUT/mame" -nvram_directory nvram-usf2
  MAME displays the Hyper Street Fighter II description and checksum warnings.
  Accept the warnings to play Ultra Street Fighter II. A frontend may hide
  sets that fail its ROM audit; use the command above in that case.
  The original USF2 program, graphics, key and sound data are preserved.
  The compact audio is mirrored to fit the existing loader's sample space.

HBMAME
  The custom usf2t8/usf2j8 sets require matching compiled definitions.
  In a clean HBMAME source checkout, install them before compiling:
    python3 install_driver.py --driver PATH_TO_CPS2_CPP --list PATH_TO_GAME_LIST
  Use src/hbmame/drivers/cps2.cpp and src/hbmame/hbmame.lst. Both the
  C++ definitions and game-list registration are installed together. Existing
  conflicting USF2 definitions fail instead of being overwritten.
  Then build HBMAME and launch by usf2t8/usf2j8. Definitions for all four
  native profiles are included for custom emulator users.
  Emulators use native QSound music; CPK playback is a MiSTer CPS+ feature.

MUSIC ONLY
  python3 music.py --nsp USA_BASE.nsp --keys prod.keys \
    --work music-scratch --out usf2_arrange.cpk
  The normal MiSTer/all build does this automatically. No existing CPK is an
  input shortcut. Tracks, conversion, loop points and trigger mapping must
  reproduce the qualified arrangement exactly.

VERIFY AFTER MOVING THE OUTPUT
  python3 output/verify_output.py --root output

RECONSTRUCTION FORMAT
  Unchanged bytes come from canonical arcade inputs. Changed/added bytes are
  source-relative XOR deltas, compressed and masked using actual decrypted
  NSP resource bytes with separate payload identities. This includes authored
  and embedded donor art and the frozen converted game resources. The public
  download contains no usable artwork masters. A published input checksum is
  not a decryption key. This is an input dependency, not redistribution control
  over the files someone has already reconstructed.

  This frozen installer does not rerun the development artwork pipeline or
  artistic tools. The CPK is freshly generated from NSP music. Every result
  is checked against the selected native candidate before publication.

SOURCE AND LICENSES
  core/SOURCE_AND_LICENSE.txt and cps2_core_release.json identify the approved
  CPS+ binary, corresponding source revision and licenses. The bundled source
  tools carry their original notices. Do not distribute this kit together
  with reconstructed game archives, NSPs, keys or generated copyrighted audio.

CREDITS
  Akuma's imported artwork is Tyrone's, from Super Street Fighter II Turbo: New Legacy.
