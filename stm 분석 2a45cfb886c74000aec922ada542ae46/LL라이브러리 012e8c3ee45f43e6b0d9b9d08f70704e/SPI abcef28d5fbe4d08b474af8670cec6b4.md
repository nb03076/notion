# SPI

```c
typedef struct
{
  uint32_t TransferDirection;       /*!< Specifies the SPI unidirectional or bidirectional data mode.
                                         This parameter can be a value of @ref SPI_LL_EC_TRANSFER_MODE.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetTransferDirection().*/

  uint32_t Mode;                    /*!< Specifies the SPI mode (Master/Slave).
                                         This parameter can be a value of @ref SPI_LL_EC_MODE.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetMode().*/

  uint32_t DataWidth;               /*!< Specifies the SPI data width.
                                         This parameter can be a value of @ref SPI_LL_EC_DATAWIDTH.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetDataWidth().*/

  uint32_t ClockPolarity;           /*!< Specifies the serial clock steady state.
                                         This parameter can be a value of @ref SPI_LL_EC_POLARITY.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetClockPolarity().*/

  uint32_t ClockPhase;              /*!< Specifies the clock active edge for the bit capture.
                                         This parameter can be a value of @ref SPI_LL_EC_PHASE.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetClockPhase().*/

  uint32_t NSS;                     /*!< Specifies whether the NSS signal is managed by hardware (NSS pin) or by software using the SSI bit.
                                         This parameter can be a value of @ref SPI_LL_EC_NSS_MODE.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetNSSMode().*/

  uint32_t BaudRate;                /*!< Specifies the BaudRate prescaler value which will be used to configure the transmit and receive SCK clock.
                                         This parameter can be a value of @ref SPI_LL_EC_BAUDRATEPRESCALER.
                                         @note The communication clock is derived from the master clock. The slave clock does not need to be set.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetBaudRatePrescaler().*/

  uint32_t BitOrder;                /*!< Specifies whether data transfers start from MSB or LSB bit.
                                         This parameter can be a value of @ref SPI_LL_EC_BIT_ORDER.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetTransferBitOrder().*/

  uint32_t CRCCalculation;          /*!< Specifies if the CRC calculation is enabled or not.
                                         This parameter can be a value of @ref SPI_LL_EC_CRC_CALCULATION.

                                         This feature can be modified afterwards using unitary functions @ref LL_SPI_EnableCRC() and @ref LL_SPI_DisableCRC().*/

  uint32_t CRCPoly;                 /*!< Specifies the polynomial used for the CRC calculation.
                                         This parameter must be a number between Min_Data = 0x00 and Max_Data = 0xFFFF.

                                         This feature can be modified afterwards using unitary function @ref LL_SPI_SetCRCPolynomial().*/

} LL_SPI_InitTypeDef;

LL_SPI_TransmitData8
LL_SPI_ReceiveData8
```