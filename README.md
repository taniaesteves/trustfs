# TrustFS
(c) 2019 INESC TEC.  Written by T. Esteves, R. Macedo and J. Paulo.

Based on SafeFS written by J. Paulo, R. Pontes and D. Burihabwa.

TrustFS is an open-source programmable, and modular stackable file system framework for implementing secure content-aware storage functionalities over hardware-assisted trusted execution environments.  TrustFS extends the original SafeFS architecture to provide the isolated exe- cution guarantees of Intel SGX. 

For more information regarding SafeFS you may read the paper published in SYSTOR'17:

- "SafeFS: A Modular Architecture for Secure User-Space File Systems"

## TrustFS Layers and Drivers:

Similar to SafeFS, in folder conf/ there are several .ini files with examples on how layers and drivers can be stacked. As an example, the aligned_det.ini file has the following configuration:

A multiple backend layer (multi_loop) is at the end of the stack (value 0), while an encryption layer (sfuse) is stacked on top of it (value 1). Then, a block virtualization layer (block_align) is stacked on top (value 2) of the encryption layer. If one wants to use just an encryption layer, then only this layer must be defined in the configuration file. 

After defining the order by which layers are stacked, we must define, for each layer, the specific settings. 

The multiple_backend layer is writing data to a single storage backend that, in this example is a local machine folder (path_1 = /srv/nfsclient). The FUSE filesystem is mounted on another machine folder (root = /mnt/point). In this configuration data is being written to a single backend.

The encryption layer is using AES (mode = 3). The key, iv, iv-size and key-size values are also specified in the configuration file. Finally, since FUSE can issue read and write requests for a dynamic number of bytes and AES is being used as a block cipher, the block virtualization layer is translating Fuse requests into blocks (mode = 2) with a size of 4096 bytes. Note that bottom layers will only require this abstaction when these layers use block-oriented techniques (erasure codes, standard/deterministic block encryption, etc).

Next we detail all the parameters for the configuration file.

Layers ([layers])

- multi_loop (position)
- sfuse (position)
- block_align (position)
- dedup (position)
- compress (position)

Multiple backend configuration ([multi_loop]):

- mode: replication (0), XOR (1), erasure codes (2)
- root: Path to the folder where the filesystem will be mounted
- ndevs: number of devices where data will be stored. E.g., if a size two is chosen for mode 0 (replication), then data will be replicated in two devices.
- path_*: path for devices where data will be stored. E.g., if ndevs has value two then a path_1 and path_2 must be assigned.

Encryption layer configuration ([sfuse]):

- mode: without encryption (0), without encryption with padding (1), standard encryption (2), deterministic encryption (3), random authentication encryption (4). Option 1 is used for testing the overhead of having a layer that adds a padding to data without encrypting it.
- key: cipher key for standard and deterministic encryption schemes.
- key_size: key size for standard and deterministic encryption schemes.
- iv: initialization vector for deterministic encryption.
- iv_size: iv size for standard and deterministic encryption schemes.
- operation_mode: cipher mode: CBC (1) or CTR (2) for deterministic encryption, and GCM (1) or CCM (2) for random authentication encryption. 

Block virtualization layer ([block_align]):

- mode: do not create virtual blocks (0), create a block abstraction layer for subsequent layers (1).
- block_size: size of the block created with the abstraction.

Deduplication layer ([dedup]):

- impl: deduplication implementation: DdumbFS (0) or LessFS (1)
- hash: hash algorithm: SHA1, TIGER128, TIGER160, TIGER
- format: if set to 1 will format the root directory.
- partition_size = size of the partition to the deduplication system.
- block_size: size of the block to be analysed.
- root_path: path to the root directory
- mode: do not create virtual blocks (0), create a block abstractio layer for subsequent layers (1).
- block_size: size of the block created with the abstraction.

Compression layer ([compress]):

- impl: compression implementation: Fusecompress (0)
- path: path to the root directory
- alg: compression algorithm: lzo
- trusted: if set to 1 will use the enclave to compress/decompress the data
- cipher_mode: encryption mode used to cipher the data
- cipher_blocksize: block size of the encrypted data
- compress_blocksize: block size to be used at compression

## Compiling TrustFS

TrustFS currently relies on several projects:

* libfuse
* libgcrypt
* libglib2.0
* libssl
* automake
* libtool
* libmhash
* libglib2.0
* libgcrypt
* libdb
* libtokyocabinet-dev
* libnfs
* liberasurecode (1.2)
* zlog (1.2.12)
* liblzo2-2
* liblzo2-dev


On ubuntu, some of these dependencies can be installed using the following command:
```bash
apt-get install automake libtool libfuse-dev libgcrypt-dev libglib2.0-dev libssl-dev libmhash-dev libdb-dev libnfs-dev libtokyocabinet-dev liblzo2-2 liblzo2-dev
```
The last two libraries have to be manually compiled and installed (ie: `make install`).
The expected version of [liberasurecode](https://github.com/openstack/liberasurecode) can be found [here](https://github.com/openstack/liberasurecode/releases/tag/1.2.0).
Same for [zlog](https://github.com/HardySimpson/zlog) where an installable version can be found [here](https://github.com/HardySimpson/zlog/releases/tag/1.2.12).

In order to be able to execute TrustFS with SGX, it will be necessary to install the Intel(R) SGX SDK and Intel(R) SGX PSW. The expected version of the [Intel SGX](https://github.com/intel/linux-sgx) can be found [here](https://github.com/intel/linux-sgx/releases/tag/sgx_2.4).

Once the dependencies are installed, place yourself at the root of the project and run:
```bash
./autogen.sh
./configure
make
sudo make install
```
The command above should produce a binary named `sfs` that will be used to start the file system.

## Running TrustFS

To run locally, compile the code as described in the previous section and run the following command: 
```bash
./sfs /mnt/point -oallow_other conf=conf_examples/default.ini
```
where `/mnt/point` is the mount point for your instance of SafeFS. `conf_examples/default.ini` is the configuration file safefs will load. If the argument is not specified the default one will be loaded.

##### 
For more information please contact:

Tânia Esteves: tania.c.araujo at inesctec.pt

Ricardo Macedo: ricardo.g.macedo at inesctec.pt

João Paulo: jtpaulo at inesctec.pt
