# 第一题：编写存证模块的单元测试代码，包括：
https://github.com/arielwang829/substrate-node-template/blob/main/substrate-node-template/pallets/poe/src/tests.rs
## 创建存证的测试用例

```
#[test]
fn create_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];
        assert_ok!(PoeModule::create_claim(Origin::signed(1), claim.clone()));
        assert_eq!(
            Proofs::<Test>::get(&claim),
            (1u64, frame_system::Pallet::<Test>::block_number())
        );
    })
}

#[test]
fn create_claim_failed_when_claim_already_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());
        assert_noop!(
            PoeModule::create_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::ProofAlreadyClaimed
        );
    })
}
```

## 撤销存证的测试用例

```
#[test]
fn revoke_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_ok!(PoeModule::revoke_claim(Origin::signed(1), claim.clone()));
        // assert_eq!(Proofs::<Test>::get(&claim), None);
        assert_eq!(PoeModule::proofs(&claim), (0, 0));
    })
}

#[test]
fn revoke_claim_failed_when_claim_is_not_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];

        // 返回错误，对链上状态不进行任何修改
        assert_noop!(
            PoeModule::revoke_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::NoSuchProof
        );
    })
}

// 作业1：撤销存证不是本人
#[test]
fn revoke_claim_failed_when_not_claim_owner() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_noop!(
            PoeModule::revoke_claim(Origin::signed(2), claim.clone()),
            Error::<Test>::NotProofOwner
        );
    })
}
```

## 转移存证的测试用例

```
// 作业2：成功转移存证
#[test]
fn transfer_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![1, 2, 3, 4];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_ok!(PoeModule::transfer_claim(
            Origin::signed(1),
            claim.clone(),
            2
        ));
        assert_eq!(
            Proofs::<Test>::get(&claim),
            (2u64, frame_system::Module::<Test>::block_number())
        );
    })
}

// 作业3：转移存证失败=>存证不存在
#[test]
fn transfer_claim_failed_when_claim_is_not_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        assert_noop!(
            PoeModule::transfer_claim(Origin::signed(1), claim.clone(), 2),
            Error::<Test>::NoSuchProof
        );
    })
}

// 作业4：转移存证失败=>存证非所有者
#[test]
fn transfer_claim_failed_with_wrong_owner() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_noop!(
            PoeModule::transfer_claim(Origin::signed(2), claim.clone(), 3),
            Error::<Test>::NotProofOwner
        );
    })
}
```

# 第二题：创建存证时，为存证内容的哈希值 Vec：
https://github.com/arielwang829/substrate-node-template/blob/main/substrate-node-template/pallets/poe/src/tests.rs
## 设置长度上限，超过限制时返回错误

-
```
pallets/poe/src/mock.rs
```
```
parameter_types! {
    pub const MaxClaimLength: u32 = 5;
}

impl pallet_poe::Config for Test {
    type Event = Event;

    type MaxClaimLength = MaxClaimLength;
}
```
-
```
pallets/poe/src/lib.rs
```
```
   #[pallet::config]
    pub trait Config: frame_system::Config {
        /// Because this pallet emits events, it depends on the runtime's definition of an event.
        type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

        type MaxClaimLength: Get<u32>;
    }


    #[pallet::call]
    impl<T: Config> Pallet<T> {
        #[pallet::weight(1_000)]
        pub(super) fn create_claim(
            origin: OriginFor<T>,
            proof: Vec<u8>,
        ) -> DispatchResultWithPostInfo {
            let sender = ensure_signed(origin)?;

            ensure!(
                T::MaxClaimLength::get() >= proof.len() as u32,
                Error::<T>::ProofTooLong
            );
            
            ....

            Ok(().into())
        }
```

## 并编写测试用例

```
// 作业5：存证内容过长异常
#[test]
fn create_claim_failed_when_claim_too_long() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1, 2, 3, 4, 5];
        assert_noop!(
            PoeModule::create_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::ProofTooLong,
        );
    });
}
```

# 测试结果
![](https://i.ibb.co/QJ8zZBr/2021-08-26-5-23-58.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
