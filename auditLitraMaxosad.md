Litra APP
=============
Project overview

the connection between some files is not visible. Multiple addresses have the power to break a contracts. 
Finding Severity breakdown

All vulnerabilities discovered during the audit are classified based on their potential severity and have the following classification:
Severity | Description
-- | ---
Critical | Bugs leading to assets theft, fund access locking, or any other loss funds to be transferred to any party.
High     | Bugs that can trigger a contract failure. Further recovery is possible only by manual modification of the contract state or replacement.
Medium   | Bugs that can break the intended contract logic or expose it to DoS attacks, but do not cause direct loss funds.
Findings

### Critical
#### 1 UNVERIFIED_TOKEN_ADDRESS
##### Description
File ``SimpleBurner.sol``, 26 line 
function ``burn`` has parameter ``address _wnft``
the name indicates that the expected address is wrapped nft
but we can pass the address of any other token even ours.
where we can ``mint`` to us a lot of tokens and in line 44 of this pool, we call ``exchange``
 and make an approve to the ``SimpleBurn`` address. I didn’t find implementation of functions 
``find_pool_for_coins, coins and exchange``
That is why I am just guessing.
if line 30 ``find_pool_for_coins`` taking variable weth, judging by the name, this is the address of wrapped eth
So we get a ``pool`` from the transferred token and wrapped eth and in line 44 of this ``pool``, we call ``exchange``. 
That will mean that ``SimpleBurner.sol`` will swap our newly created unsecured tokens on weth and lose money
##### Recommendation
pass address wrapped nft via constructor and use it 
delete ``burn`` ``address _wnft`` parameter
####2 RISK_OF_BLURRING
##### Description
File ``La.sol``, 151 line 
if one person takes or group of people possession of minter, they could mint token only to their addresses thereby redistributing the liquidity of this token in their favor. for example, there were 100 tokens that cost 100 ether, we issued ourselves another 100, now these 200 tokens account for 100 ether, the value of the token has fallen by 2 times customers have lost value. blurring can occur not only when minter is 1 person, but also when it is a group of people who were able to negotiate and master minter. 
The main component of blurring is that minter can ``mint`` tokens almost for free, just call function.
Other problem is only minter can ``mint``. if he decides not to issue tokens anymore, then other users will not be able to receive them, i.e. the opportunity to enter the project will be closed, for example, to other investors. customers will miss out on potential benefits

same in ``WrappedNFT.sol`` line 11 ``creator = msg.sender``
##### Recommendation
delete minter 
make person pay for minting, ``mint`` token to msg.sender address
####3 CONSTANT_RECEIVER
##### Description
File ``SimpleBurner.sol``, 21 line We set ``receiver``. we use it we use it in 
44 line ``pool.exchange`` as a parameter, Judging by the name of variable, this is the one who should receive funds by burning their tokens. but ``receiver`` was set in ``constructor`` and there is no functionality to change it. So It could be all profits from burs goes to one person. In that way customers nothing left ot just transfer their funds to reciver . But it could be just fee if function ``pool.exchange`` make transfer to tx.origin and transfer some fee to ``receiver``.
##### Recommendation
Implement function ``exchange`` that way:make transfer to tx.origin and transfer some fee to receiver.
####4 AVAILABLE_SUPPLY_IS_NOT_CHECKED_WHEN_MINTING_TOKENS
##### Description
File ``La.sol``, 151 line 
according to the idea of the token, it cannot be issued more than the allowed amount. and every epoch this allowed amount expands. There are no checks for theavailable supply in ``mint``. So theoretically we can issue a lot of tokens. This means that the shares of the holders of this token will be blurred, which means they will lose funds. 
##### Recommendation
add check 
```
require (this.totalSupply() + value <= availiableSupply(), “not allowed to mint such amount”);
```
### High
#### 1 STOP_THE_BURNER
##### Description	
File ``SimpleBurner.sol`` extends from ``Stoppable.sol`` 
and on line 26 function ``burn`` has ``modifier onlyNotStopped``
``modifier onlyNotStopped`` checks if variable ``stopped`` is equal to false ``emergencyAdmin, ownershipAdmin`` has power to set ``stopped`` to true

If all burners will be stopped then users' funds will be stuck on the contract
##### Recommendation
There are few ways things could be done to avoid freezing funds.
``emergencyAdmin, ownershipAdmin`` should not be externally owned address. It should be contract controlled by many interested people. This solution will save you from admin who doing anything to robber your money. This solution satisfies the Byzantine fault tolerance system. Files ``EmergencyAdminManaged.sol, OwnershipAdminManaged.sol`` there are opportunity to choose new ``emergencyAdmin and ownershipAdmin``, if it will be externally owned address the danger of freezing will appear again. But we relay responsibility to interested people of our current ``emergencyAdmin, ownershipAdmin``
add additional checks which will not allow to block the burner if there are no alternatives. There is a possibility that the remaining burner is unprofitable, but users' funds will not be completely stuck.
#### 2  EXCHANGE_DOES_NOT_RECEIVE_ETHER_ALTHOUGH_IT_REQUIRES
##### Description	
File ``ICurve.sol ``
line 4 function ``exchange`` is ``payable``
parameter 5 bool ``use_eth``
File ``SimpleBurner.sol`` line 44 the fifth parameter set to`` true`` judging by the name of the fifth parameter, this function with fifth parameter true use ether. but the ether is not transmitted
##### Recommendation
it is necessary to add the logic of calculating the amount
```
pool.exchange{value: amount}(i, j, inAmount, 0, true, receiver);
```
#### 3 RECEIVER_COULD_HAVE_BEEN_BLACKLISTED
##### Description
File ``SimpleBurner.sol``
``receiver`` was set in ``constructor`` and there is no functionality to change it. 
If ``receiver`` by some reason can't get what's being sent to him by ``exchange``  function, then all ``exchange`` calls will revert. So funds will be unburnable.

same in ``WrappedNFT.sol`` line 11 ``creator = msg.sender``
##### Recommendation
add function that could change ``receiver`` 
#### 4 ASSIGNED_ZERO_TOKEN_ADDRESS
##### Description
File ``Voting.sol`` line 123
we can call initialize with ``_token`` = ``address(0)`` and whole contract will be ruined, because we won't be able to change it
##### Recommendation
add 
```
require(_token != address(0), “zero address”);
```



### Medium
#### 1 OUTDATED_AVAILIABLESUPPLY
##### Description
``LA `` 85 lines
``startEpochSupply`` may be irrelevant. I.e. ``_updateMiningParamters()`` has not been called yet in this epoch; 
we can break the logic of programs that rely on this function
##### Recommendation
``availiableSupply`` is a view function, which means that we cannot change storage, that means, we cannot update the ``startEpochSupply`` variable. Therefore, the only thing we can do is not to issue an irrelevant value.
add check 
```
require(block.timestamp < startEpochTime + RATE_REDUCTION_TIME, “not actual startEpochSupply, call updateMiningParamters()”);
```
#### 2 OUTDATED_AVAILIABLESUPPLY
##### Description
``LA``  98 lines
``startEpochTime`` may be irrelevant. I.e.``_updateMiningParamters()`` has not been called yet in this epoch; we can break the logic of programs that rely on this function
##### Recommendation
```
require(block.timestamp < startEpochTime + RATE_REDUCTION_TIME, “not actual startEpochTime, call updateMiningParamters()”);
```
#### 3 CURRENT_RATE_IS_NOT_CONSIDERED_CORRECTLY
##### Description
``LA``  131 lines
In the ``mintableInTimeframe`` function, we go from the end to the beginning and each time we try to restore the previous ``rate``. but we will not be able to do this correctly in principle.
going forward , we calculated the ``rate`` using the following formula
we can break the logic of programs that rely on this function
```
currentRate = currentRate * RATE_DENOMINATOR / RATE_REDUCTION_COEFFICIENT;
```
and going back , we try to calculate the ``rate`` using the following formula
```
currentRate = currentRate * RATE_REDUCTION_COEFFICIENT / 
RATE_DENOMINATOR; 
```
consider the example 
currentRate =2 
RATE_DENOMINATOR = 3
RATE_REDUCTION_COEFFICIENT = 5
currentRate * RATE_DENOMINATOR/RATE_REDUCTION_COEFFICIENT = 2 * 3 / 5 = 1 
because we have integer division
now let's try to recover the old rate
currentRate =1
RATE_DENOMINATOR = 3
RATE_REDUCTION_COEFFICIENT = 5
 currentRate * RATE_REDUCTION_COEFFICIENT/RATE_DENOMINATOR = 1 * 5 / 3 = 1
we didn't get back the ``rate`` we had
all because of integer division
This problem is serious I don’t think we can recover old ``rate ``
##### Recommendation
therefore, I propose to abandon the passage from the end to the beginning. Choose another one. Calculate the ``rate`` from ``INITIAL_RATE`` to start and go from start to end, accumulating supply and updating the ``rate`` every new epoch
### Low
#### 1 INCORRECT_ERROR_MESSAGE 
##### Description
File ``EmergencyAdminManaged.sol``, 23 line 
Required ``msg.sender`` equal to future ``EmergencyAdmin`` but if not we get error with comment `` “! emergency admin”``. ``EmergencyAdmin`` and ``future Emergency Admin`` is two different person and this comment misleads the user.
##### Recommendation
change ``"! emergency admin"`` to ``"! future emergency admin"``
#### 2 DIFFERENT_ERROR_MESSAGE_TO_SIMILAR_SITUATIONS
##### Description
File ``EmergencyAdminManaged.sol``, 23 line 
File ``OwnershipAdminManaged.sol``, 21 line 
File ``ParameterAdminManaged.sol``, 23 line
There are checks that ``msg.sender`` is future … admin, but in ``ParameterAdminManaged`` and ``OwnershipAdminManaged`` error message is ``"Access denied!"``  and in ``EmergencyAdminManaged`` error message is ``"! emergency admin"``
##### Recommendation
In File ``ParameterAdminManaged.so``l, 23 line replace ``"Access denied!"`` to ``“!future parameter admin”``
In File ``OwnershipAdminManaged.sol``, 21 line replace ``"Access denied!"`` to ``“!future ownership admin”``
it will also make messages more informative
#### 3 POSSIBLE_TO_CHANGE_THE_VALUE_OF_STOPPED_TO_THE_SAME
##### Description
File ``Stoppable.sol``, 14 line 
function ``setStopped`` set stopped to given value. So it could be that ``stopped`` is already true and we call ``setStopped(true)`` value will be set and we will pay gas for that, but nothing real changed
##### Recommendation
change this function to flipStopped. It would be same but with 
```
require(_stopped != stopped, “already this value”);
```
#### 4 VARIABLE_TOKEN0_USED_ONE_TIME
##### Description
``FileSimpleBurner.sol``, 34-35 lines
the variable ``token0`` is used only 1 time after creation. we can replace variable to its value and save gas
##### Recommendation
```
if(pool.coins(0) == _wnft) {
```
#### 5 OPERATIONS_WHOSE_RESULT_CAN_BE_PRECALCULATED
##### Description
``LA.sol``
10 line
```
uint256 constant private YEAR = 86400 * 365;
```
14 line
```
uint256 constant private INITIAL_RATE = 100638977635782747603833865 / YEAR;
```
17 line
```
uint256 constant private RATE_REDUCTION_COEFFICIENT = 1252 * 1e15;
```
18 line
```
uint256 constant private RATE_DENOMINATOR = 10 ** 18;
```
34line      
```
startEpochTime = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME;
```
there are operation between constant values. We can calculate so as not to waste extra gas
##### Recommendation 
```
uint256 constant private YEAR = 3024000;
```
```
uint256 constant private INITIAL_RATE = 3191241046289407268
```
```
uint256 constant private RATE_REDUCTION_COEFFICIENT = 1252000000000000000;
```
```
uint256 constant private RATE_DENOMINATOR = 1000000000000000000;
```
```
startEpochTime = block.timestamp -31449600;
```
#### 6 TWO_SAME_CONSTANT_VARIABLE
##### Description
``LA`` 15 line
```
uint256 constant private RATE_REDUCTION_TIME = YEAR;
```
Two same constant variable. We wasting gas on assignment and creation Constant in storage
##### Recommendation
replace all ``RATE_REDUCTION_TIME`` to ``YEAR`` and delete 15 line

#### 7 STORED_CONSTANT_VARIABLE_WHICH_IS_USED_ONE_TIME_IN_CONSTRUCTOR
##### Description
``LA ``19 line
variable ``INFLATION_DELAY``  used one time in constructor but we spend gas to save it in storage
##### Recommendation
delete line 19 
pass this variable value via constructor parameter
line 30 
```
constructor(uint inflationDelay) ERC20('Litra Token', 'LA') {
```
#### 8 NEW_VARIABLE_EQUAL_TO_CONSTANT_DOESNТT_CHANGING
##### Description
``LA`` 31 line
variable ``initSupply`` equal to  ``INITIAL_SUPPLY`` and does not change in life, we waste gas to create this variable and to set it value;
##### Recommendation
delete line 31
replace variable  ``initSupply`` to variable ``initialSupply``, witch passed via constructor parameter
#### 9 ASSIGNMENT_CONSTANT_VALUE_TO_VARIABLE_IN_CONSTRUCTOR
##### Description
``LA`` 35-36 lines
```
variable miningEpoch = -1;
rate = 0;
```
could win in gas 
##### Recommendation
move assign from constructor to variable declaration
#### 10 ASSIGNMENT_SAME_VALUE
##### Description
``LA`` 53 
     ``startEpochSupply = _startEpochSupply;``
when ``rate = 0``, ``_startEpochSupply`` does not change, and writing to storage is an expensive operation. We're wasting gas
##### Recommendation
do not assign ``startEpochSupply`` when ``rate = 0``
move assign else if branch
#### 11 EXTRA_VARIABLE
##### Description
``LA`` 41-42 
     ``_startEpochSupply`` and ``_rate`` could be avoided 
and we will not waste gas on is’s creation and assignment

##### Recommendation

delete 41 42
lines 46 - 54 replace with 
```
if(rate == 0) {
rate =Initial_Rate;
} else { 
startEpochSupply += rate*RATE_REDUCTION_TIME;
rate = rate * RATE_DENOMINATOR / RATE_REDUCTION_COEFFICIENT;
}
delete line 53 54

```
#### 12 THE__STARTEPOCHTIME_VARIABLE_COPIES_THE_STARTEPOCHTIME_VARIABLE
##### Description
``LA``  65, 75 line 
the ``_startEpochTime``  variable is assigned the value of the ``startEpochTime`` variable and then these variables do not change in any way. So we waste gas on creation variable ``_startEpochTime`` and assign value to it
##### Recommendation
delete lines 65,75
replace all ``_startEpochTime`` on ``startEpochTime``
#### 13 DELETE_THE_ELSE_BRANCH_AND_TAKE_OUT_THE_GENERAL_CODE
##### Description
``LA``  68-71, 78-81 line 
either if is executed and we change ``_startEpochTime`` to the current one, or we already had it up-to-date. In any case, the current value will be in ``startEpochTime``. 	We delete else branch codesize could become less so we could save some gas
##### Recommendation
delete lines  68-71 write just 
```
return startEpochTime
```
delete lines  68-71 write just 
```
return startEpochTime + RATE_REDUCTION_TIME
```
#### 14 AVAILABLESUPPLY_JUST_CALLS_ANOTHER_FUNCTION
##### Description
``LA``  88-89 lines
we waste gas for call another function. 
84 line 
Strange ``internal view`` combination.
the view was made to look at the values for free, while if this function is called from a non-view function, then we will pay
##### Recommendation
delete internal version make this function public 
replace news 84 - 90 to 
```
function availiableSupply() external view returns(uint256) {
return startEpochSupply + (block.timestamp - startEpochTime) * rate;
}
```
#### 15 TYPO_IN_THE_WORD_PARAMETER
##### Description
``LA``  40 line
function ``_updateMiningParamters`` the letter e is missing in parameters
##### Recommendation
replace all ``Paramters`` on Parameters
#### 16 COULD_MINT_ZERO_TOKENS
``LA``  151 line
we can call ``mint`` with ``_value = 0``. we spent gas but didn't get any real changes
##### Recommendation
depends on what we want
if we want to revert
add check ``value != 0 ``
```
require(_msgSender() == minter, "!minter");
```
if we want to continue to work 
then we wrap ``mint``
```
if (_value !=0) {
_mint(_to, _value);
}	
```
#### 17 TYPO_IN_THE_WORD_KECCAK
##### Description
``Voting`` 31 line
```
bytes32 public constant DISABLE_VOTE_CREATION = 0x40b01f8b31b51596de2eeab8c325ff77cc3695c1c1875d66ff31176e7148d2a1; //keccack256("DISABLE_VOTE_CREATION")
```
опечатка в слове ``keccak256``

##### Recommendation
replace`` keccack256`` with keccak256
#### 18 COMPARES_CONSTANTS_WITH_THE_VALUES_THAT_WERE_ASSIGNED_TO_THEM
##### Description
``Voting`` 134 - 140 lines    
 unnecessary superfluous asserts. we're wasting gas
##### Recommendation
delete lines 134 - 140
#### 19  MISSING_PARAM_COMMENT_FOR__MINBALANCEUPPERLIMIT
##### Description
``Voting`` after 119 line
 написаны ``param`` комментарии ко всем параметрам кроме ``_minBalanceUpperLimit``
##### Recommendation
write ``param`` commet after 119 line for ``_minBalanceUpperLimit`` parameter
#### 20 REDUNDANT_I_J_VALUE
##### Description
``SimpleBurner.sol`` 32,33 line
variables ``i`` ``j`` take the values 0 and 1 but we created them uint256 it's redundant
##### Recommendation
replace line 32, 33 
```
uint256 i;
uint256 j;
```
with minimum possible uint range
```
uint8 i;
uint8 j;
```
#### 21 DUPLICATE_CODE
##### Description
``Voting.sol`` 526-528 lines and 540-542 lines
same checks
##### Recommendation
delete one of them
#### 22 REDUNDANT_CHECK
##### Description
``Voting.sol``  526, 530 lines
line 531 function ``_canExecute`` returns ``false``  when ``vote_.execute``
line 527 function ``_canExecute`` returns ``false``  when 
``getTimestamp64() < vote_.startDate.add(voteTime) && !vote_.executed``

if ``vote_.execute`` = ``true`` then ``_canExecute`` returns ``false``
if ``vote_.execute`` = ``false`` then ``_canExecute`` value depends from
``getTimestamp64() < vote_.startDate.add(voteTime)``
!vote_.executed check is redundant 
##### Recommendation
delete ``&& !vote_.executed`` from ``_isVoteOpen`` line 569
#### 23 ALWAYS_RETURN_TRUE
##### Description
``Voting.sol`` 322 line
function ``isFrowarder`` always return ``true``
##### Recommendation
replace all ``isFrowarder()`` with ``true``
#### 24 COULD_BE_EXTERNAL
##### Description
``Voting.sol`` 
355 line ``canExecute``
365 line ``canVote``
387 line ``getVote``
423 line ``getVoterState``
could be move from public to external to save gas
##### Recommendation
could be move from public to external to save gas
#### 25 TYPO_IN_THE_WORD_AVAILIABLE
##### Description
``La ``88 line
```
function availiableSupply() external view returns(uint256) {
```
##### Recommendation
replace all ``availiable`` with available
#### 26 TODO_COMMENT
##### Description
``Voting.sol`` 513 line
TODO commet
##### Recommendation
finish what you was going to do






